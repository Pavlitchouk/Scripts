# Powershell script for creating alerts with log analytics

This is a script which allows you to create alerts based on log analytics info that's pulled into the workspace.

Make sure to have a log analytics workspace created, and take note of what subscription, resource group its in.

You also need to take note of the action group, as this script assumes you have created this. I suppose you can change it to a logic app if you require.


```
#Script created by Andrei Pavlitchouk
#You will need two modules installed on your powershell install-module az.monitor and install-module Az.OperationalInsights
#Remove this if you like, just will execute and force you to connect-azureAD
Connect-AzureAD
#Change if you want subscription to be in a different stack
Select-AzSubscription -SubscriptionName "yourloganalyticssubscription"
#Change these to be your log analytics
$LogAnalyticsResourceGroup= "yourloganalyticsresourcegroup"
$workspaceName= "yourloganalyticsname"
#Don't change this
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName $LogAnalyticsResourceGroup -Name $workspaceName
#Enter computer name into the values, please note, this is case sensitive.
$dimension = New-AzMetricAlertRuleV2DimensionSelection -DimensionName "Computer" -ValuesToInclude "yourcomputername" 
#Enter the threshold, you will need to change the metricname to whatever you want. Please note this has to match exactly as your create a manual alert
#Change Threshold if you want a different CPU %
$criteria = New-AzMetricAlertRuleV2Criteria -MetricName "Average_% Processor Time" `
-DimensionSelection $dimension `
-TimeAggregation Average `
-Operator  GreaterThanOrEqual `
-threshold 70 `
#Set the actiongroup at the very end
$act = Get-AzActionGroup -ResourceGroupName "Default-ActivityLogAlerts" -Name "youractiongroupname"
$action = New-AzActionGroup -ActionGroupId $act.id
#Set the severity level
$severity = "3"
#Change the windows size
$windowsize = "00:15:00"
#Use this to change the frequency
$frequency = "00:05:00"
#Sets the alert name
$alertname = "youralertname"
Add-AzMetricAlertRuleV2 -Name $alertname `
-ResourceGroupName $LogAnalyticsResourceGroup `
-WindowSize $windowsize `
-Frequency $frequency `
-TargetResourceId $workspace.ResourceId `
-Condition $criteria `
-Severity $severity `
-ActionGroup $action
```