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
<img src="testcmvmss.PNG" width="250" height="300" />
<img src="enableinsightVMSS.PNG" width="300" height="250" />
<img src="extensionistalled.PNG" width="300" height="250" />
      

2. Enable Certain event Data to be pushed from the Agent on Log analytics workspace
  <img src="advancedsettings.PNG" width="300" height="250" />
  
   1. Click on Data --> Windows Event Logs --> on the box type Application(I am pushing an application type event from windows VM)
   
   2. Click on Save
      1. Please note: any VM with an MMA agent that is connected to the same workspace will now report any windows event of the type Application to the log analytics Event Table along with any other Event type that is being specified.
<img src="eventtype.PNG" width="300" height="250" />

3. Create custom autoscale Scale out rule based on an event metric and enable diagnostic settings to query "AutoscaleEvaluationsLog" and "AutoscaleScaleActionsLog "  [ the metric criteria as Source = "testCMtrigger"]
  1. please modify the rule as per the highlighted field
  2. the autoscale rule will check for an event METRIC (not an event - will explain later ) that has the source value of "testCMtrigger", it will check the count every 1 minute , and if within the last 2 minutes it is greater than 1 , it will trigger the autoscale.
<img src="createcustomScaleoutarule.PNG" width="300" height="250" />
<img src="rule1.PNG" width="300" height="250" />
<img src="rule2.PNG" width="300" height="250" />

4. Push an event from within the windows VM with a specific Source "testCMtrigger"




