---
title: 跨区域移动 Azure 资源的支持
description: 列出可跨 Azure 区域移动的 Azure 资源类型
ms.service: azure-resource-manager
ms.topic: reference
origin.date: 08/25/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: yes
ms.testdate: 08/24/2020
ms.author: v-yeche
ms.openlocfilehash: 96ad47ac29c901d0a13e0e221517d5196c32e63b
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92469963"
---
<!--Verify Successfully-->
# <a name="support-for-moving-azure-resources-across-regions"></a>跨区域移动 Azure 资源的支持

本文确认了是否支持将某种 Azure 资源类型移到另一个 Azure 区域。 

跳转到资源提供程序命名空间：
> [!div class="op_single_selector"]
> - [Microsoft.AAD](#microsoftaad)
> - [microsoft.aadiam](#microsoftaadiam)
> - [Microsoft.AlertsManagement](#microsoftalertsmanagement)
> - [Microsoft.AnalysisServices](#microsoftanalysisservices)
> - [Microsoft.ApiManagement](#microsoftapimanagement)
> - [Microsoft.Authorization](#microsoftauthorization)
> - [Microsoft.Automation](#microsoftautomation)
> - [Microsoft.AzureActiveDirectory](#microsoftazureactivedirectory)
> - [Microsoft.AzureStack](#microsoftazurestack)
> - [Microsoft.Batch](#microsoftbatch)
> - [Microsoft.Blueprint](#microsoftblueprint)
> - [Microsoft.Cache](#microsoftcache)
> - [Microsoft.Cdn](#microsoftcdn)
> - [Microsoft.ClassicCompute](#microsoftclassiccompute)
> - [Microsoft.ClassicNetwork](#microsoftclassicnetwork)
> - [Microsoft.ClassicStorage](#microsoftclassicstorage)
> - [Microsoft.CognitiveServices](#microsoftcognitiveservices)
> - [Microsoft.Compute](#microsoftcompute)
> - [Microsoft.ContainerInstance](#microsoftcontainerinstance)
> - [Microsoft.ContainerRegistry](#microsoftcontainerregistry)
> - [Microsoft.ContainerService](#microsoftcontainerservice)
> - [Microsoft.DataBox](#microsoftdatabox)
> - [Microsoft.DataFactory](#microsoftdatafactory)
> - [Microsoft.DataMigration](#microsoftdatamigration)
> - [Microsoft.DBforMariaDB](#microsoftdbformariadb)
> - [Microsoft.DBforMySQL](#microsoftdbformysql)
> - [Microsoft.DBforPostgreSQL](#microsoftdbforpostgresql)
> - [Microsoft.Devices](#microsoftdevices)
> - [Microsoft.DocumentDB](#microsoftdocumentdb)
> - [Microsoft.EventGrid](#microsofteventgrid)
> - [Microsoft.EventHub](#microsofteventhub)
> - [Microsoft.HDInsight](#microsofthdinsight)
> - [Microsoft.ImportExport](#microsoftimportexport)
> - [microsoft.insights](#microsoftinsights)
> - [Microsoft.IoTCentral](#microsoftiotcentral)
> - [Microsoft.KeyVault](#microsoftkeyvault)
> - [Microsoft.Kusto](#microsoftkusto)
> - [Microsoft.Logic](#microsoftlogic)
> - [Microsoft.MachineLearningServices](#microsoftmachinelearningservices)
> - [Microsoft.ManagedIdentity](#microsoftmanagedidentity)
> - [Microsoft.Media](#microsoftmedia)
> - [Microsoft.Network](#microsoftnetwork)
> - [Microsoft.NotificationHubs](#microsoftnotificationhubs)
> - [Microsoft.OperationalInsights](#microsoftoperationalinsights)
> - [Microsoft.OperationsManagement](#microsoftoperationsmanagement)
> - [Microsoft.Portal](#microsoftportal)
> - [Microsoft.PowerBI](#microsoftpowerbi)
> - [Microsoft.PowerBIDedicated](#microsoftpowerbidedicated)
> - [Microsoft.RecoveryServices](#microsoftrecoveryservices)
> - [Microsoft.Relay](#microsoftrelay)
> - [Microsoft.ResourceGraph](#microsoftresourcegraph)
> - [Microsoft.Scheduler](#microsoftscheduler)
> - [Microsoft.Search](#microsoftsearch)
> - [Microsoft.Security](#microsoftsecurity)
> - [Microsoft.ServiceBus](#microsoftservicebus)
> - [Microsoft.ServiceFabric](#microsoftservicefabric)
> - [Microsoft.SignalRService](#microsoftsignalrservice)
> - [Microsoft.Solutions](#microsoftsolutions)
> - [Microsoft.Sql](#microsoftsql)
> - [Microsoft.Storage](#microsoftstorage)
> - [Microsoft.StreamAnalytics](#microsoftstreamanalytics)
> - [Microsoft.TimeSeriesInsights](#microsofttimeseriesinsights)
> - [Microsoft.Web](#microsoftweb)

## <a name="microsoftaad"></a>Microsoft.AAD

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- | 
> | domainservices | 否 | 
> | domainservices/replicasets | 否 | 

## <a name="microsoftaadiam"></a>microsoft.aadiam

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | tenants | 否 |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | actionrules | 否 | 

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | servers | 否 |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | 服务 |  是（使用模板） <br/><br/> [跨区域移动 API 管理](../../api-management/api-management-howto-migrate.md)。 | 

<!--Not Available on ## Microsoft.AppConfiguration-->
<!--Not Available on ## Microsoft.AppService-->

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | policyassignments | 否 |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | automationaccounts | 是（使用模板） <br/><br/> [使用异地复制](../../automation/automation-managing-data.md#geo-replication-in-azure-automation) |  
> | automationaccounts/configurations | 否 | 
> | automationaccounts/runbooks | 否 | 

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | b2cdirectories | 否 | 

<!--Not Available on ## Microsoft.AzureData-->

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | registrations | 否 | 

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | batchaccounts |  Batch 帐户不能直接从一个区域移到另一个区域，但你可以使用模板来导出模板，对其进行修改，然后将模板部署到新区域。 <br/><br/> 了解如何[跨区域移动 Batch 帐户](../../batch/best-practices.md#moving-batch-accounts-across-regions) |
<!--Not Available on ## Microsoft.BatchAI-->
<!--Not Available on ## Microsoft.BingMaps-->
<!--Not Available on ## Microsoft.BizTalkServices-->
<!--Not Available on ## Microsoft.Blockchain-->

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | blueprintassignments | 否 | 

<!--Not Available on ## Microsoft.BotService-->

## <a name="microsoftcache"></a>Microsoft.Cache

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | redis | 否 | 

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | cdnwebapplicationfirewallpolicies | 否 |
> | 配置文件 | 否 | 
> | profiles/endpoints | 否 | 

<!--Not Available on ## Microsoft.CertificateRegistration-->

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | domainnames | 不会为经典服务计划任何工作。
> | virtualmachines | 否 | 

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | networksecuritygroups | 不会为经典服务计划任何工作。
> | reservedips | 否 | 
> | virtualnetworks | 否 | 

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | storageaccounts | 是 |  

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | accounts | 否 | 

<!--Not Available on Learn about [moving your Azure Cognitive Search service to another region](../../search/search-howto-move-across-regions.md)-->

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | availabilitysets | 是 | 
> | diskencryptionsets | 否 | 
> | disks | 是 | 
> | galleries | 否 | 
> | galleries/images | 否 | 
> | galleries/images/versions | 否 | 
> | hostgroups | 否 | 
> | hostgroups/hosts | 否 | 
> | images | 否 | 
> | proximityplacementgroups | 否 | 
> | restorepointcollections | 否 | 
> | sharedvmimages | 否 | 
> | sharedvmimages/versions | 否 | 
> | snapshots | 否 | 
> | virtualmachines | 是 | 
> | virtualmachines/extensions | 否 | 
> | virtualmachinescalesets | 否 | 

<!--Not Available on ## Microsoft.Container-->

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | containergroups | 否 | 

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | registries | 否 |  
> | registries/buildtasks | 否 |  
> | registries/replications | 否 | 
> | registries/tasks | 否 |  
> | registries/webhooks | 否 | 

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | containerservices | 否。<br/><br/> 服务[已停用](https://azure.microsoft.com/updates/azure-container-service-will-retire-on-january-31-2020/)。
> | managedclusters | 否 | 
> | openshiftmanagedclusters | 否 | 

<!--Not Available on ## Microsoft.ContentModerator-->
<!--Not Available on ## Microsoft.CortanaAnalytics-->
<!--Not Available on ## Microsoft.CostManagement-->
<!--Not Available on ## Microsoft.CustomerInsights-->
<!--Not Available on ## Microsoft.CustomProviders-->

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | jobs | 否 | 

<!--Not Available on ## Microsoft.DataBoxEdge-->
<!--Not Available on ## Microsoft.Databricks-->
<!--Not Available on ## Microsoft.DataCatalog-->
<!--Not Available on ## Microsoft.DataConnect-->
<!--Not Available on ## Microsoft.DataExchange-->

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | datafactories | 否 | 
> | factories | 否 |  

<!--Not Available on ## Microsoft.DataLake-->
<!--Not Available on ## Microsoft.DataLakeAnalytics-->
<!--Not Available on ## Microsoft.DataLakeStore-->

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | services | 否 | 
> | services/projects | 否 | 
> | slots | 否 | 

<!--Not Available on ## Microsoft.DataShare-->

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | servers | 可以使用跨区域只读副本移动现有服务器。 [了解详细信息](../../postgresql/howto-move-regions-portal.md)。<br/><br/> 如果为服务预配了异地冗余备份存储，则可以使用异地还原在其他区域中进行还原。 [了解详细信息](../../mariadb/concepts-business-continuity.md#recover-from-an-azure-regional-data-center-outage)。

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | servers | 可以使用跨区域只读副本移动现有服务器。 [了解详细信息](../../mysql/howto-move-regions-portal.md)。

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | servergroups | 否 | 
> | servers | 可以使用跨区域只读副本移动现有服务器。 [了解详细信息](../../postgresql/howto-move-regions-portal.md)。
> | serversv2 | 否 | 

<!--Not Available on ## Microsoft.DeploymentManager-->

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | elasticpools | 否。 资源未公开。
> | elasticpools/iothubtenants | 否。 资源未公开。
> | iothubs | 是的。 [了解详细信息](../../iot-hub/iot-hub-how-to-clone.md)
> | provisioningservices | 否 | 

<!--Not Available on ## Microsoft.DevSpaces-->
<!--Not Available on ## Microsoft.DevTestLab-->

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | databaseaccounts | 否 | 

<!--Not Available on ## Microsoft.DomainRegistration-->
<!--Not Available on ## Microsoft.EnterpriseKnowledgeGraph-->

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | domains | 否 |  
> | topics | 否 | 

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | clusters | 否 |  
> | namespaces | 是（带模板）<br/><br/> [将事件中心命名空间移到另一个区域](../../event-hubs/move-across-regions.md) | 

<!--Not Available on ## Microsoft.Genomics-->
<!--Not Available on ## Microsoft.HanaOnAzure-->

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | clusters | 否 | 

<!--Not Available on ## Microsoft.HealthcareApis-->
<!--Not Available on ## Microsoft.HybridCompute-->
<!--Not Available on ## Microsoft.HybridData-->

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | jobs |  否 | 

## <a name="microsoftinsights"></a>microsoft.insights

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | accounts | 否。 [了解详细信息](../../azure-monitor/faq.md#how-do-i-move-an-application-insights-resource-to-a-new-region)。
> | actiongroups |  否 | 
> | activitylogalerts | 否 | 
> | alertrules |  否 | 
> | autoscalesettings |  否 | 
> | components |  否 |  
> | guestdiagnosticsettings | 否 | 
> | metricalerts | 否 | 
> | notificationgroups | 否 | 
> | notificationrules | 否 | 
> | scheduledqueryrules |  否 | 
> | webtests |  否 | 
> | workbooks |  否 |  

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | checknameavailability |  否。<br/><br/> IoT Central 使用地理位置而不是区域。
> | graph | 否

<!--Not Available on ## Microsoft.IoTHub-->
<!--Not Available on ## Microsoft.IoTSpaces-->

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | hsmpools | 否 | 
> | vaults |  否 | 

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | clusters |  否 |  

<!--Not Available on ## Microsoft.LabServices-->
<!--Not Available on ## Microsoft.LocationBasedServices-->
<!--Not Available on ## Microsoft.LocationServices-->

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | hostingenvironments | 否 | 
> | integrationaccounts |  否 |  
> | integrationserviceenvironments | 否 | 
> | isolatedenvironments | 否 | 
> | workflows |  否 |  

<!--Not Available on ## Microsoft.MachineLearning-->
<!--Not Available on ## Microsoft.MachineLearningCompute-->
<!--Not Available on ## Microsoft.MachineLearningExperimentation-->
<!--Not Available on ## Microsoft.MachineLearningModelManagement-->
<!--Not Available on ## Microsoft.MachineLearningOperationalization-->

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | workspaces | 否 | 

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | userassignedidentities | 否 | 

<!--Not Available on ## Microsoft.Maps-->
<!--Not Available on ## Microsoft.MarketplaceApps-->

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | mediaservices |  否 | 
> | mediaservices/liveevents |  否 | 
> | mediaservices/streamingendpoints |  否 | 

<!--Not Available on ## Microsoft.Microservices4Spring-->
<!--Not Available on ## Microsoft.Migrate-->
<!--Not Available on ## Microsoft.NetApp-->
<!--DDos, Front Door, and Private link service are not available on Mooncake-->
<!--MOONCAKE CUSTOMIZEIONT-->

## <a name="microsoftnetwork"></a>Microsoft.Network

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | applicationgateways | 否 | 
> | applicationgatewaywebapplicationfirewallpolicies | 否 | 
> | applicationsecuritygroups |  否 |  
> | azurefirewalls |  否 |  
> | bastionhosts | 否 | 
> | connections |  否 | 
> | dnszones |  否 | 
> | expressroutecircuits | 否 | 
> | expressroutecrossconnections | 否 | 
> | expressroutegateways | 否 | 
> | expressrouteports | 否 | 
> | loadbalancers | 是 <br/><br/> 可以将现有配置导出为模板，然后在新区域中部署该模板。 了解如何移动[外部](../../load-balancer/move-across-regions-external-load-balancer-portal.md)或[内部](../../load-balancer/move-across-regions-internal-load-balancer-portal.md)负载均衡器。 |
> | localnetworkgateways |  否 | 
> | natgateways |  否 | 
> | networkintentpolicies |  否 | 
> | networkinterfaces | 是 | 
> | networkprofiles | 否 | 
> | networksecuritygroups | 是 | 
> | networkwatchers |  否 |  
> | networkwatchers/connectionmonitors |  否 | 
> | networkwatchers/lenses |  否 | 
> | networkwatchers/pingmeshes |  否 | 
> | p2svpngateways | 否 | 
> | privatednszones |  否 |  
> | privatednszones/virtualnetworklinks |  否 |  
> | privateendpoints | 否 | 
> | privatelinkservices | 否 | 
> | publicipaddresses | 是<br/><br/> 可以将现有公共 IP 地址配置导出为模板，然后在新区域中部署该模板。 [详细了解](../../virtual-network/move-across-regions-publicip-portal.md)如何移动公共 IP 地址。 |
> | publicipprefixes | 否 | 
> | routefilters | 否 | 
> | routetables |  否 | 
> | serviceendpointpolicies |  否 | 
> | trafficmanagerprofiles |  否 | 
> | virtualhubs | 否 | 
> | virtualnetworkgateways |  否 |  
> | virtualnetworks |  否 | 
> | virtualnetworktaps | 否 | 
> | virtualwans | 否 | 
> | vpngateways（虚拟 WAN） | 否 | 
> | vpnsites（虚拟 WAN） | 否 | 
> | webapplicationfirewallpolicies |  否 | 

<!--DDos, Front Door, and Private link service are not available on Mooncake-->
<!--MOONCAKE CUSTOMIZEIONT-->

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | namespaces |  否 | 
> | namespaces/notificationhubs |  否 |  

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | workspaces |  否 | 

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | managementconfigurations |  否 | 
> | 视图 |  否 | 

<!--Not Available on ## Microsoft.Peering-->

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | dashboards | 否 | 

<!--Not Available on ## Microsoft.PortalSdk-->

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | workspacecollections |  否 | 

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | capacities |  否 | 

<!--Not Available on ## Microsoft.ProjectOxford-->

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | vaults | 否。<br/><br/> 不支持跨 Azure 区域移动 Azure 备份的恢复服务保管库。<br/><br/> | 

<!--Not Available on [disable and recreate the vault](../../site-recovery/move-vaults-across-regions.md)-->

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | namespaces |  否 | 

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | 查询 |  否 |  

<!--Not Available on ## Microsoft.Resources-->
<!--Not Available on deploymentScripts-->
<!--Not Available on templateSpecs-->

<!--Not Available on ## Microsoft.SaaS-->

## <a name="microsoftscheduler"></a>Microsoft.Scheduler

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | flows |  否 |  
> | jobcollections |  否 | 

## <a name="microsoftsearch"></a>Microsoft.Search

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | searchservices |  否 | 

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | iotsecuritysolutions |  否 | 
> | playbookconfigurations | 否 | 

<!--Not Available on ## Microsoft.ServerManagement-->

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | namespaces |  否 | 

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | applications | 否 | 
> | clusters |  否 | 
> | clusters/applications | 否 | 
> | containergroups | 否 | 
> | containergroupsets | 否 | 
> | edgeclusters | 否 | 
> | networks | 否 | 
> | secretstores | 否 | 
> | volumes | 否 | 

<!--Not Available on ## Microsoft.ServiceFabricMesh-->

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | signalr |  否 |  

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | appliancedefinitions | 否 | 
> | appliances | 否 | 
> | applicationdefinitions | 否 | 
> | applications | 否 | 
> | jitrequests | 否 | 

## <a name="microsoftsql"></a>Microsoft.Sql

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | instancepools | 否 | 
> | managedinstances | 是 <br/><br/> [详细了解](../../azure-sql/database/move-resources-across-regions.md)如何在区域之间移动托管实例。 | 
> | managedinstances/databases | 是 | 
> | servers | 是 | 
> | servers/databases | 是 <br/><br/> [详细了解](../../azure-sql/database/move-resources-across-regions.md)如何在区域之间移动数据库。<br/><br/>   | 
> | servers/elasticpools | 是 <br/><br/> [详细了解](../../azure-sql/database/move-resources-across-regions.md)如何在区域之间移动弹性池。<br/><br/>  | 
> | virtualclusters | 是 | 

<!--MOONCAKE: Not Available on [Learn more](../../resource-mover/tutorial-move-region-sql.md)-->

<!--Not Available on ## Microsoft.SqlVirtualMachine-->
<!--Not Available on ## Microsoft.SqlVM-->

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | storageaccounts | 是<br/><br/> [将 Azure 存储帐户移到另一个区域](../../storage/common/storage-account-move.md) | 

<!--Not Available on ## Microsoft.StorageCache-->
<!--Not Available on ## Microsoft.StorageSync-->
<!--Not Available on ## Microsoft.StorageSyncDev-->
<!--Not Available on ## Microsoft.StorageSyncInt-->
<!--Not Available on ## Microsoft.StorSimple-->

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | streamingjobs |  否 |  

<!--Not Available on ## Microsoft.StreamAnalyticsExplorer-->
<!--Not Available on ## Microsoft.TerraformOSS-->

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | environments |  否 | 
> | environments / eventsources |  否 |  
> | environments/referencedatasets |  否 | 

<!--Not Available on ## Microsoft.Token-->
<!--Not Available on ## Microsoft.VirtualMachineImages-->
<!--Not Available on ## microsoft.visualstudio-->
<!--Not Available on ## Microsoft.VMwareCloudSimple-->

## <a name="microsoftweb"></a>Microsoft.Web

> [!div class="mx-tableFixed"]
> | 资源类型 | 区域移动 | 
> | ------------- | ----------- |
> | certificates | 否 | 
> | connectiongateways |  否 |  
> | connections |  否 |  
> | customapis |  否 | 
> | hostingenvironments | 否 | 
> | serverfarms |  否 |  
> | sites |  否 | 
> | sites/premieraddons |  否 |  
> | sites/slots |  否 |  

<!--Not Available on ## Microsoft.WindowsIoT-->
<!--Not Available on ## Microsoft.WindowsVirtualDesktop-->

## <a name="third-party-services"></a>第三方服务

第三方服务目前不支持移动操作。

<!-- Update_Description: update meta properties, wording update, update link -->
