# Azure Idle/Unused Resource Advisor - Sample KQLs

These KQLs have been added into Workbook to capture the unused resources runnings on your cloud Infrastructure. You can review the criteria and remediation recommendations for each resource type.


##  ApplicationGateway

Application gateways are quite expensive resource in Azure and identifying an Unused Application gateways is going to be an effective cost optimization effort.

####  Criteria:
*  Application gateways without any backend pool targets configured.
*  Application gateways has unhealthy backend and No success/Direction traffics
*  Resource shouldnt be built recently 


#### AppGw - without Backendpool Targets
```
resources
| join (
    resources
    | where type =~ 'microsoft.network/applicationgateways'
    | mv-expand backendConfig=properties.backendAddressPools
    | summarize TotalBackendCount=count(backendConfig) by name ) on name
| join (
    resources
    | where type =~ 'microsoft.network/applicationgateways'
    | mv-expand Config=properties.backendAddressPools
    | extend emptypoolconfig = Config.properties.backendIPConfigurations
    | where isnull(emptypoolconfig)
    | where Config.properties.backendAddresses == "[]" and properties.redirectConfigurations == "[]"
    | summarize TargetCount=count(Config.properties.backendAddresses) by name ) on name
| where TargetCount == TotalBackendCount
| where type =~ 'Microsoft.Network/applicationGateways'
| where properties.operationalState != 'Stopped'
| extend Details = pack_all()
| project Resource=id, resourceGroup, location, subscriptionId, tags, Details

```

#### Note
If you have identified any Unused AppGW using the above KQL query, need to ensure the same status was persisted for atleast 180 days by running the below Kql in Log Analytics Workspace. 

```
AzureDiagnostics
| where ResourceType == "APPLICATIONGATEWAYS" and OperationName == "ApplicationGatewayAccess"
| where TimeGenerated > ago(180d)
| extend code = case( httpStatus_d between (200 .. 299), "2xx", httpStatus_d between (300 .. 399), "3xx",  httpStatus_d between (400 .. 499), "4xx" , httpStatus_d between (500 .. 599), "5xx", "NULL")
| summarize AggregatedValue = count() by code,  ResourceId, Resource, ResourceGroup, SubscriptionId 

```
Output should not have any 2xx/3xx httpcodes logs reported as below image.

![image](https://github.com/azure-scavengers/Azure-Unused-Orphan-CostOptimization/blob/ce8541b3cef3605511cf8cf6b2c79894c7611684/Docs/AppGwStatus.png)

#### Remediations:

*  Aggressive -  Immediate Decommissioning of the unused application gateway and associated resources (Public IP/Certificates)
*  Conservative - Unused Application gateways can be stopped using the following PS commands which stops the incurring cost to zero.
   
   ```
   $AppGw = Get-AzApplicationGateway -Name "AppGatewayname" -ResourceGroupName "AppgwRSGgroup"
   Stop-AzApplicationGateway -ApplicationGateway $AppGw
   ```

## App Service Plans

Azure AppService plans are the other expensive resources to be considered for cost optimization. ASP cost is based on the provisioned SKU and Instances count

Criteria:

*  ASP without any Application Hosted 
*  ASP with application hosted but stopped in state (webApp/Api/Slots)
*  Resource shouldnt be built recently
*  ASP with Only Functionapp having NO Function ExecutionCounts

#### ASP - without Apps

```
resources
| where type =~ "microsoft.web/serverfarms"
| where sku.name <> "F1" or sku.name <> "Y1"
| where properties.numberOfSites == 0 and properties.currentNumberOfWorkers != 0
| extend Details = pack_all()
| project Resource=id, resourceGroup, location, subscriptionId, Sku=sku.name, Tier=sku.tier, tags ,Details

```

####  ASP - with App stopped
```
resources
| where type =~ "microsoft.web/serverfarms"      
| where sku.name <> "F1"
| where properties.kind <> "functionapp" 
| extend NoOfApps= properties.numberOfSites
| where NoOfApps != 0
| join kind=leftouter ( 
    resources
    | where type =~ "Microsoft.Web/sites" 
    | extend Appstate=properties.state
    | where Appstate == "Stopped"  
    | extend serverFarmId=split(properties.serverFarmId, '/')[-1]   
    | summarize AppCount=count(Appstate) by tostring(serverFarmId), location ) on location        
| where name == serverFarmId
| where NoOfApps == AppCount
| extend Details = pack_all()
| project Resource=id, resourceGroup, location, subscriptionId, Sku=sku.name, Tier=sku.tier, NoOfApps, AppCount, tags, Details

```

#### Note
If you have identified any Unused ASPs using the above KQL query, need to ensure the same status was persisted for atleast 180 days by running the below Kql in Log Analytics Workspace. You should not have any logs reported in that output.

```
AzureMetrics
| where ResourceProvider =~"microsoft.web"
| where TimeGenerated > ago(180d)
```
####  ASP - with Only FunctionApps
```
resources
| where type =~ "microsoft.web/serverfarms"
| where tostring(sku.tier) <> "Dynamic" and tostring(sku.tier) <> "WorkflowStandard" and tostring(sku.tier) <> "ElasticPremium"
| extend NoOfApps= properties.numberOfSites
| where NoOfApps != 0
| join kind=leftouter (
    resources
    | where type =~ "Microsoft.Web/sites"
    | extend serverFarmId=split(properties.serverFarmId, '/')[-1]
    | where kind contains "functionapp"
    | summarize AppCount=count() by tostring(serverFarmId), Kind=kind, location) on location        
| where name == serverFarmId
| where NoOfApps == AppCount   
| extend Details = pack_all()
| project Resource=id, resourceGroup, location, subscriptionId, Kind, Sku=sku.name, Tier=sku.tier, NoOfApps, AppCount, tags, Details
```

#### Note
If you have identified any FunctionApp ASPs without any functions configured or without any Function Execution counts, Its the candidate for Decommissioning.

Use this powershell script to get the Function execution metrics. 

```
Powershell: Get-AzMetric -ResourceId "test" -MetricName "FunctionExecutionCount" -AggregationType Maximum
```
#### Remediations:

*  Aggressive -  Immediate Decommissioning of the AppService Plan ( after taking application backup)
*  Conservative - AppService plan can be scaled down to its least plan (free)


## Disks -Unattached past 30 days

```
advisorresources
| where type == 'microsoft.advisor/recommendations'
| where properties.category =~ 'Cost'
| extend solution = tostring(properties.shortDescription.solution)
| where solution contains "attached"
| extend 
    resourceID = tostring(properties.resourceMetadata.resourceId),
    sku = tostring(properties.extendedProperties._skuToken),
    lastupdated = properties.lastUpdated
| join kind=leftouter (
   resources
   | where type == "microsoft.compute/disks"  
   | extend diskState = tostring(properties.diskState)
   | where managedBy == "" and diskState != 'ActiveSAS' or diskState == 'Unattached' and diskState != 'ActiveSAS'
   | where name notcontains "ASRReplica"
   | project Diskid=['id'], Diskname=name, DiskSize= properties.diskSizeGB, CreationDate=properties.timeCreated, skuname=sku.name, subscriptionId) on subscriptionId
| where Diskid =~ resourceID 
| project resourceID, resourceGroup, subscriptionId, name, DiskSize, skuname, CreationDate

```


## Public IPs
```
Resources
| where type == "microsoft.network/publicipaddresses"
| where properties.ipConfiguration == "" and properties.natGateway == "" and properties.publicIPPrefix == "" and properties.dnsSettings == ""
| where  isnull(properties.dnsSettings)
| extend Details = pack_all()
| project Resource=id, resourceGroup, location, subscriptionId, sku.name, tags, Details
```


## Load Balancers
```
resources
| where type == "microsoft.network/loadbalancers"
| where properties.backendAddressPools == "[]"
| extend Details = pack_all()
| project subscriptionId, Resource=id, resourceGroup, location, tags, Details
```


        
## Front Door WAF Policy
```
resources
| where type == "microsoft.network/frontdoorwebapplicationfirewallpolicies"
| where properties.frontendEndpointLinks== "[]" and properties.securityPolicyLinks == "[]"
| extend Details = pack_all()
| project Resource=id, resourceGroup, location, subscriptionId, Sku=sku.name, tags, Details
```
        
## Traffic Manager Profiles
```
resources
| where type == "microsoft.network/trafficmanagerprofiles"
| where properties.endpoints == "[]"
| extend Details = pack_all()
| project Resource=id, resourceGroup, location, subscriptionId, tags, Details
```
