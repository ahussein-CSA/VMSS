# Purpose
The purpose of this guide is to setup a custom metric rule **_(a specific windows eventlog is being pushed to event table on log analytics workspace where the Microsoft Monitoring Agent (MMA) installed on the VM is connected to)_** - through log analytics workspace metric (Event) to trigger an Azure Virtual Machine Scaleset (VMSS) to scale out.

This guide does not discuss the Use host-based metrics that VMSS rule can scale based on with no customization required , for such guide please go to:  https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview#use-host-based-metrics, for this guide I will use Azure portal - ARM template guide to follow.

  **_Please note: You can use syslog event instead of windows event, for this guide I will use windows event)_**
  
  # Pre-Requisites:
  
  1. Azure Vitrual Machine Scaleset
     1. Existing or new
    
  2. Log analytics Workspace
  
  3. Windows VM: this could be an instance within the VMSS, a different Azure VM or an on-premise VM (agent must be installed on on-premise)
   1. This VM is where the event will be pushed into windows event log to be sent to the azure log analytics workspace Event table.
  
  4. MMA agent extension to be enabled on the Windows VM
     1. As pointed above, if the VM is in Azure , either within VMSS or a different Azure VM , the MMA can be installed through the extension blade.
     2. For linux , Linux diagnostic agent needs to be installed.
     
  5. Enable insight (Diagnostic Settings on the Scaleset):
    1. This will allow us to trace the autoscale evaluation logs and autoscale Actions logs- this step I recommend for troubleshooting if needed.
    

# Steps:

1. Deploy VMSS scaleset
2. Enable Insight 

![ScreenShot](LAAdvancedSettings.PNG?raw=true "LA Data settings")




