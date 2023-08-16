# Azure Unused Resource Finder

The Azure Unused Resource Finder shortly known as "Scavengers" is designed to help Azure cloud consumers to find the cost saving opportunities of their Infrastructure. It can perform assessment of multiple subscriptions at a time on different tenants of your Azure resources. It works based on Microsoft's cost optimization best practices defined in the Well-Architected and Cloud Adoption Framework.

Assess your Azure resource types for cost optimization scoped to subscriptions, management groups, or the entire tenant. Get an efficiency score for each subscription and can also provide total potential cost saving opportunity. Leverage cost data to understand optimization potential and can also provide different types of safe and secure remediation possibilities . Also it Incorporates the Azure Advisor recommendations.


Key Benefits
* Cost Optimization
* Maximizes cloud resource utilization by identifying unused or misconfigured resources.
* Cloud Governance
* Safe and Secure Remediation methods

## Table of Contents
* [Assessment Scope](#assessmentscope)
* [Workbook View](#workbookview)
* [Setting up Workbook ](#workbookcreation)
* [More Cost saving Resource Types](#ConsultingRequest)

## <a id="assessmentscope"></a> Assessment Scope

Type of resources covered in workbook:
* Application Gateway
   - AppGateways without any BackendPool Target   
* AppService Plan
   - ASPs with No applications
   - ASPs with WebApp/APi/Slots Stopped state(more than 90 Days)
*  Disk
   - Unattached Disk more than 30 days
   - Disk associated in Deallocated VMs  
* Load Balancers
   - LB without any BackendAddressPools
* Front Door WAF Policy
   -  FD without any FrontendEndpointlinks/securityPolicyLinks
* Traffic Manager Profiles
   - TM without any Endpoints
* Public IPs
  - PIP without any IPconfigured


## <a id="ConsultingRequest"></a> More Cost saving Resource Types

Mainly focussed on the costly resource types using KQL queries and PowerShell scripts for the Idle Utilization and misconfigurations. 
*  PowerBI Embedded Capacity
*  SQL DBs/Managed Instances (single)
*  SQL DBs (ElasticPools)
*  CosmosDB

On demand Cost optimizing automations using azure functions are available based on resource Utilization.

<b>[Looking for a Demo? Connect Us!](https://airtable.com/appSMMDDdyPWKvAaC/shrrdqfA5X775v2gq)</b>

## <a id="workbookview"></a> Azure Workbook Offerings

This Azure workbook designed to showcase the Unused or Idle resources running on your environment, also shows the provisioned SKU/Capacities and current state of the resources with the complete lists. You can download the inventory from this workbook and take necessary remediation accordingly. [Analysis Method and Remediation recommendations](https://github.com/azure-scavengers/Azure-Unused-Resources/blob/3d5b34a428e8c6133a76f2594751ceb6d312a10b/Sample-KQLs/Idle-resources-SampleKQLs.md) 

Dashboard view contains multiple tabs with relevant details

![image](https://github.com/azure-scavengers/Azure-Unused-Orphan-CostOptimization/blob/ce8541b3cef3605511cf8cf6b2c79894c7611684/Docs/workbook-Overview.png)
![image](https://github.com/azure-scavengers/Azure-Unused-Orphan-CostOptimization/blob/ce8541b3cef3605511cf8cf6b2c79894c7611684/Docs/AppGw-workbook.png)


All the information presented in this Workbook is based on Azure Resource Graph queries.
<img src="https://user-images.githubusercontent.com/69309933/172938464-38b08c8e-0d4d-493b-aa8f-954189556d7a.png" width="20" height="20">

## <a id="workbookcreation"></a> Setting up Workbook 
Importing this Workbook to your Azure environment is quite simple.

Follow this steps to use the Workbook:
* Login to [Azure Portal](https://portal.azure.com/) <img src="https://user-images.githubusercontent.com/69309933/172941966-9e030031-6ccb-4ebf-bd2b-04bb623e5ff7.png" width="20" height="20">
* Go to _'Azure Workbooks'_

<img src="https://user-images.githubusercontent.com/69309933/172806635-14051976-328e-4623-96ab-0dd6a7bc7817.png" width="350">

* Click on _'+ Create'_

<img src="https://user-images.githubusercontent.com/69309933/172807465-cced3466-0669-423b-87b3-8fa70fdbf1d1.png" width="350">

* Click on _'+ New'_

<img src="https://user-images.githubusercontent.com/69309933/172807547-52d790ce-7852-4b4b-a81f-81e8b7fac26e.png" width="350"> 

* Open the Advanced Editor using the _'</>'_ button on the toolbar

<img src="https://user-images.githubusercontent.com/69309933/172807673-dfc63741-0c40-47c0-ab58-d39309b06e69.png" width="700"> 

* Select the _'Gallery Template'_ (step 1)
* Replace the JSON in the gallery template to the [Idle Resource template](https://github.com/azure-scavengers/Azure-Unused-Resources/blob/main/Workbook/Workbook.json) (step 2)
* Click _'Apply'_ (step 3)

<img src="https://user-images.githubusercontent.com/69309933/172807762-17aec6f9-4a81-4d5b-9017-673a0ab6b26e.png" width="700"> 

* Click in the ‘Save’ button on the toolbar

<img src="https://user-images.githubusercontent.com/69309933/172807909-b4527207-343e-4861-af4e-35e1104029d1.png" width="700">  

* Select a name and where to save the Workbook:
- Title: _'Orphan Resources'_
- Subscription: _Subscription Name_
- Resource group: _Resource Group Name_
- Location: _Region_
* Click _'Save'_
  
<img src="https://user-images.githubusercontent.com/69309933/172808030-3d7171c9-8b23-4f69-ab8b-7150b1459ea8.png" width="700">  
The Workbook is ready to use!
* Click on _'Workbooks'_
* Click on _'Orphan Resources'_ Workbook.

<img src="https://user-images.githubusercontent.com/69309933/172808358-ed2fede8-42a4-42bd-9c68-3ac4d645f812.png" width="700">  

Start using the Workbook and review your orphan resources.<br/>
Filter by specific subscription is optional.

![image](https://github.com/azure-scavengers/Azure-Unused-Orphan-CostOptimization/blob/ce8541b3cef3605511cf8cf6b2c79894c7611684/Docs/workbook-Overview.png)





