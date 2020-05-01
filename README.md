# Creating a custom metric rule to scale out Azure Virtual Machine Scaleset (VMSS) using custom Events other than using Application insight metrics.


This document is intended to provide guidance on how to scale out an azure virtual machine scaleset based on a custom metric that is triggered by an event pushed from a virtual machine to log analytics workspace.

The main guide is [Here](VMSScustomMetricLogAnalytics.md)

This guide is performed using Azure Potral , An ARM template guide will follow

## Conclusion:

This guide is not using the applicaiton insight metrics to trigger VMSS autoscale. it uses a custom metric/event for this.

If you are interested to use application insight to perform the same, please check the below 2 links: 
Here](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview#application-level-metrics-with-app-insights) 
<br> And 
[Here](https://docs.microsoft.com/en-us/azure/azure-monitor/app/java-in-process-agent)

