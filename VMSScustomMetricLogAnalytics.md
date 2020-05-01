# Purpose
The purpose of this guide is to setup a custom metric rule **_(a specific windows eventlog is being pushed to event table on log analytics workspace where the Microsoft Monitoring Agent (MMA) installed on the VM is connected to)_** - through log analytics workspace metric (Event) to trigger an Azure Virtual Machine Scaleset (VMSS) to scale out.

This guide does not discuss the Use host-based metrics that VMSS rule can scale based on with no customization required , for such guide please go to:  https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview#use-host-based-metrics, for this guide I will use Azure portal - ARM template guide to follow.

  **_Please note: You can use syslog event instead of windows event, for this guide I will use windows event)_**
  
  # Pre-Requisites:
  
  1. Azure Vitrual Machine Scaleset
     1. Existing or new
     2. Install MMA extension - you can do that by enabling insight from within VMSS
    
  2. Log analytics Workspace
  
  3. Windows VM: this could be an instance within the VMSS, a different Azure VM or an on-premise VM (agent must be installed on on-premise)
   1. This VM is where the event will be pushed into windows event log to be sent to the azure log analytics workspace Event table.
  
  4. MMA agent extension to be enabled on the Windows VM
     1. As pointed above, if the VM is in Azure , either within VMSS or a different Azure VM , the MMA can be installed through the extension blade.
     2. For linux , Linux diagnostic agent needs to be installed.
     
  5. Enable insight (Diagnostic Settings on the Scaleset):
    1. This will allow us to trace the autoscale evaluation logs and autoscale Actions logs- this step I recommend for troubleshooting if needed.
    

# Steps:

1. Deploy VMSS scaleset:
   1. I have create a VMSS "testCMVMSS"with Instance Count 2 and insight is enabled to push some data for troubleshooting
   2. Install MMA extension on the VMSS (this can be by deploying the MMA extension , or by enabling the insight)
      1. This will help for tracking the "AutoscaleEvaluationsLog" and "AutoscaleScaleActionsLog " 
      2. Please note: the above step is necessary if the event will be triggered from within the VMSS then the MMA need to be installed, however in here I use it to give you an insight of how the scaling is triggered, you can disable it at later stage.
      3. I am using a different VM to trigger the Event.
      
<img src="testcmvmss.PNG"/><br>

<img src="enableinsightVMSS.PNG" /><br>

<img src="extensionistalled.PNG" /><br>
      

2. Enable Certain event Data to be pushed from the Agent on Log analytics workspace
  <img src="advancedsettings.PNG" />
  
   1. Click on Data --> Windows Event Logs --> on the box type Application(I am pushing an application type event from windows VM)
   
   2. Click on Save
      1. Please note: any VM with an MMA agent that is connected to the same workspace will now report any windows event of the type Application to the log analytics Event Table along with any other Event type that is being specified.
<img src="eventtype.PNG" />

3. Create custom autoscale Scale out rule based on an event metric and enable diagnostic settings to query "AutoscaleEvaluationsLog" and "AutoscaleScaleActionsLog "  [ the metric criteria as Source = "testCMtrigger"]

    1. please modify the rule as per the highlighted field
    2. the autoscale rule will check for an event METRIC (not an event - will explain later ) that has the source value of "testCMtrigger", it will check the count every 1 minute , and if within the last 2 minutes it is greater than 1 , it will trigger the autoscale. [current instance count is 2 , maximum I can get is 3]
  
<img src="createcustomScaleoutarule.PNG" /><br>

<img src="rule1.PNG" /><br>

<img src="rule2.PNG" /><br>

<img src="rule3.PNG" /><br>


4. Once the rule is saved, enable diagnostic settings to track AutoscaleEvaluationsLog" and "AutoscaleScaleActionsLog " - this doesn ot have to be same workspace you are pushing the event to - in my case I use the same


<img src="diagnosticsettings1.PNG" /><br>

<img src="diagset2.PNG" /><br>

5. Push the application event metric from with VM , then query the log analytics workspace to see the event appears in there

<img src="pushevent.PNG" /><br>

6. Running a query to check what evaluation Data is being captured.

  1. I have run the query below to see what is the current autoscale evaluation is reporting, as seen in the screenshot , metric data is empty and even though an event is being pushed but it does not appear in there 
  
  2. The reason for the above is : Event is pushed to an Event table , yet VMSS  scaleset is checking different table for a Metric called an Event.

``` Kusto Query

AutoscaleEvaluationsLog
| where TimeGenerated > ago(3m)
| where OperationName == "MetricEvaluation" or OperationName == "ScaleRuleEvaluation" 
| where ResourceId contains "84" 
| extend vmssName= split(ResourceId,"/")[8]
| project vmssName  , OperationName, MetricData, ObservedValue, Threshold, EstimateScaleResult, TimeWindow , TimeGrainStatistic , TimeGenerated | order by TimeGenerated desc

``` 
<img src="logcapture1.PNG" /><br>

7. To move the EVent from the event table to the timeseries table where VMSS read the Event metric, we will need to create an Alert on the log analytics workspace (where the event is being pushed to ) that will never fire or be triggered, yet it is a copy mechanism.


  1. Please note: for the Alert rule it is important to make the threshold less than 0 - We do not want the alert to be triggered , it is a way just to copy the event into an event metric.
  

<img src="alert1.PNG" /><br>
<img src="alert2.PNG" /><br>
<img src="alert3.PNG" /><br>
<img src="alert4.PNG" /><br>


8. Once the above is done , now you can see that there is a metric Data appears on the AutoscaleEvaluationsLog , where it mentions if a scale set is triggered or not, as the threshold for the rule is > 1 and it checks only for last 2 minutes, then no triggering 

<img src="scalerule1.PNG" /><br>
<img src="logcapture2.PNG" /><br>
<img src="logcapture3.PNG" /><br>


9- I will push more than one Event now , so the threshold for the last 2 minutes will be more than 1 then a scale out trigger is now taking place

<img src="logcapturetriggered.PNG" /><br>

10- A new instance is being created (maximum instance count is 3)

<img src="instancecreation.PNG" /><br>
<img src="instancecreation1.PNG" /><br>

11- the new instance is running. (3 instances running now , and the autoscale rule will stop as the maximum instance count is met)

<img src="instancecreated.PNG" /><br>




