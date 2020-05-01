# Creating a custom metric rule to scale out Azure Virtual Machine Scaleset (VMSS) using custom Events other than using Application insight metrics.


This document is intended to provide guidance on how to scale out an azure virtual machine scaleset based on a custom metric that is triggered by an event pushed from a virtual machine to log analytics workspace.

The main guide is [Here](VMSScustomMetricLogAnalytics.md)

This guide is performed using Azure Potral , An ARM template guide will follow

## Conclusion:

For Java The best approach instead of using custom application metrics events is to enable Austoscale based on an application insight metrics, which can be embeded within the code or codeless.
[Here](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview#application-level-metrics-with-app-insights) 
[Here](https://docs.microsoft.com/en-us/azure/azure-monitor/app/java-in-process-agent)

