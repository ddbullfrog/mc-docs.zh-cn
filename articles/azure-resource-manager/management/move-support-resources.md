---
title: 资源类型支持的移动操作
description: 列出可移到新资源组或订阅的 Azure 资源类型。
ms.topic: conceptual
origin.date: 09/23/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: yes
ms.testdate: 08/24/2020
ms.author: v-yeche
ms.openlocfilehash: 87b041da9b2d8d6d68f9de429b88b13f9450ff7d
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92469962"
---
<!--Verify Successfully-->
# <a name="move-operation-support-for-resources"></a>支持移动操作的资源

本文列出某个 Azure 资源类型是否支持移动操作。 它还提供了有关移动资源时要考虑的特殊条件的信息。

> [!IMPORTANT]
> 在大多数情况下，子资源不能独立于其父资源移动。 子资源的资源类型采用 `<resource-provider-namespace>/<parent-resource>/<child-resource>` 格式。 例如，`Microsoft.ServiceBus/namespaces/queues` 是 `Microsoft.ServiceBus/namespaces` 的子资源。 移动父资源时，子资源会随之自动移动。 如果在本文中没有看到子资源，可假定它与父资源一起移动。 如果父资源不支持移动，则无法移动子资源。

跳转到资源提供程序命名空间：
> [!div class="op_single_selector"]
> - [Microsoft.AAD](#microsoftaad)
> - [microsoft.aadiam](#microsoftaadiam)
> - [Microsoft.Advisor](#microsoftadvisor)
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
> - [Microsoft.ClassicInfrastructureMigrate](#microsoftclassicinfrastructuremigrate)
> - [Microsoft.ClassicNetwork](#microsoftclassicnetwork)
> - [Microsoft.ClassicStorage](#microsoftclassicstorage)
> - [Microsoft.ClassicSubscription](#microsoftclassicsubscription)
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
> - [Microsoft.GuestConfiguration](#microsoftguestconfiguration)
> - [Microsoft.HDInsight](#microsofthdinsight)
> - [Microsoft.ImportExport](#microsoftimportexport)
> - [microsoft.insights](#microsoftinsights)
> - [Microsoft.IoTCentral](#microsoftiotcentral)
> - [Microsoft.KeyVault](#microsoftkeyvault)
> - [Microsoft.Kusto](#microsoftkusto)
> - [Microsoft.Logic](#microsoftlogic)
> - [Microsoft.MachineLearningServices](#microsoftmachinelearningservices)
> - [Microsoft.Maintenance](#microsoftmaintenance)
> - [Microsoft.ManagedIdentity](#microsoftmanagedidentity)
> - [Microsoft.ManagedServices](#microsoftmanagedservices)
> - [Microsoft.Management](#microsoftmanagement)
> - [Microsoft.Media](#microsoftmedia)
> - [Microsoft.Network](#microsoftnetwork)
> - [Microsoft.NotificationHubs](#microsoftnotificationhubs)
> - [Microsoft.OperationalInsights](#microsoftoperationalinsights)
> - [Microsoft.OperationsManagement](#microsoftoperationsmanagement)
> - [Microsoft.PolicyInsights](#microsoftpolicyinsights)
> - [Microsoft.Portal](#microsoftportal)
> - [Microsoft.PowerBI](#microsoftpowerbi)
> - [Microsoft.PowerBIDedicated](#microsoftpowerbidedicated)
> - [Microsoft.RecoveryServices](#microsoftrecoveryservices)
> - [Microsoft.Relay](#microsoftrelay)
> - [Microsoft.ResourceGraph](#microsoftresourcegraph)
> - [Microsoft.ResourceHealth](#microsoftresourcehealth)
> - [Microsoft.Resources](#microsoftresources)
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
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | domainservices | 否 | 否 |

## <a name="microsoftaadiam"></a>microsoft.aadiam

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | diagnosticsettings | 否 | 否 |
> | diagnosticsettingscategories | 否 | 否 |
> | privatelinkforazuread | 是 | 是 |
> | tenants | 是 | 是 |

<!--Not Available on ## Microsoft.Addons-->
<!--Not Available on ## Microsoft.ADHybridHealthService-->

## <a name="microsoftadvisor"></a>Microsoft.Advisor

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | 配置 | 否 | 否 |
> | generaterecommendations | 否 | 否 |
> | metadata | 否 | 否 |
> | 建议 | 否 | 否 |
> | 禁止显示 | 否 | 否 |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | actionrules | 是 | 是 |
> | alerts | 否 | 否 |
> | alertslist | 否 | 否 |
> | alertsmetadata | 否 | 否 |
> | alertssummary | 否 | 否 |
> | alertssummarylist | 否 | 否 |
> | smartdetectoralertrules | 是 | 是 |
> | smartgroups | 否 | 否 |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | servers | 是 | 是 |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!IMPORTANT]
> 无法移动设置为消耗 SKU 的 API 管理服务。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | reportfeedback | 否 | 否 |
> | 服务 | 是 | 是 |

<!--Not Available on ## Microsoft.AppConfiguration-->
<!--Not Available on ## Microsoft.AppPlatform-->
<!--Not Available on ## Microsoft.AppService-->
<!--Not Available on ## Microsoft.Attestation-->

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | classicadministrators | 否 | 否 |
> | dataaliases | 否 | 否 |
> | denyassignments | 否 | 否 |
> | elevateaccess | 否 | 否 |
> | findorphanroleassignments | 否 | 否 |
> | 锁定 | 否 | 否 |
> | 权限 | 否 | 否 |
> | policyassignments | 否 | 否 |
> | policydefinitions | 否 | 否 |
> | policysetdefinitions | 否 | 否 |
> | privatelinkassociations | 否 | 否 |
> | resourcemanagementprivatelinks | 否 | 否 |
> | roleassignments | 否 | 否 |
> | roleassignmentsusagemetrics | 否 | 否 |
> | roledefinitions | 否 | 否 |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!IMPORTANT]
> Runbook 必须与自动化帐户存在于同一资源组中。
>

<!--Not Available on [Move your Azure Automation account to another subscription](../../automation/how-to/move-account.md?toc=/azure-resource-manager/toc.json)-->

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | automationaccounts | 是 | 是 |
> | automationaccounts/configurations | 是 | 是 |
> | automationaccounts/runbooks | 是 | 是 |

<!--Not Available on ## Microsoft.AVS-->

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | b2cdirectories | 是 | 是 |
> | b2ctenants | 否 | 否 |

<!-- Not Available on ## Microsoft.AzureData-->

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | cloudmanifestfiles | 否 | 否 |
> | registrations | 是 | 是 |

<!--Not Available on ## Microsoft.AzureStackHCI-->

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | batchaccounts | 是 | 是 |

<!-- Not Available on ## Microsoft.Billing-->
<!-- Not Available on ## Microsoft.BingMaps-->
<!-- Not Available on ## Microsoft.BizTalkServices-->
<!-- Not Available on ## Microsoft.Blockchain-->
<!-- Not Available on ## Microsoft.Blockchain-->

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | blueprintassignments | 否 | 否 |
> | blueprints | 否 | 否 |

<!-- Not Available on ## Microsoft.BotService-->

## <a name="microsoftcache"></a>Microsoft.Cache

> [!IMPORTANT]
> 如果 Azure Redis 缓存实例配置了虚拟网络，则实例无法被移动到其他订阅。 请参阅[网络移动限制](./move-limitations/networking-move-limitations.md)。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | redis | 是 | 是 |
> | redisenterprise | 否 | 否 |

<!--Not Available on ## Microsoft.Capacity-->
## <a name="microsoftcdn"></a>Microsoft.Cdn

<!--MOONCAKE CUSTOMIZATION: PROFILES AND PROFILES / ENDPOOINTS IS NO FOR SUBSCRIPTION-->
<!--UPDATE CAREFULLY, CORRECT BY EMAIL-->
<!--CORRECT ON > | profiles | Yes | No |-->
<!--CORRECT ON > | profiles / endpoints | Yes | No |-->


> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | cdnwebapplicationfirewallmanagedrulesets | 否 | 否 |
> | cdnwebapplicationfirewallpolicies | 是 | 是 |
> | edgenodes | 否 | 否 |
> | 配置文件 | 是 | 否 |
> | profiles/endpoints | 是 | 否 |

> [!NOTE]
> 目前，Azure 中国不支持有关跨订阅移动 CDN 资源的自助服务。
> 当尝试在 Azure 中国中跨订阅移动 CDN 配置文件或配置文件/终结点时，请联系 [Azure 支持](https://support.azure.cn/support/contact/)或在 [Azure 支持网站](https://support.azure.cn/support/support-azure/)上提交请求以获得帮助。
>

<!--UPDATE CAREFULLY, CORRECT BY EMAIL-->
<!--MOONCAKE CUSTOMIZATION: PROFILES AND PROFILES / ENDPOOINTS IS NO FOR SUBSCRIPTION-->

<!--Not Available on ## Microsoft.CertificateRegistration-->
<!--Not Available on ## Microsoft.ChangeAnalysis-->

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!IMPORTANT]
> 请参阅[经典部署移动指南](./move-limitations/classic-model-move-limitations.md)。 可以使用特定于该方案的操作跨订阅移动经典部署资源。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | capabilities | 否 | 否 |
> | domainnames | 是 | 否 |
> | quotas | 否 | 否 |
> | resourcetypes | 否 | 否 |
> | validatesubscriptionmoveavailability | 否 | 否 |
> | virtualmachines | 是 | 是 |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!IMPORTANT]
> 请参阅[经典部署移动指南](./move-limitations/classic-model-move-limitations.md)。 可以使用特定于该方案的操作跨订阅移动经典部署资源。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | classicinfrastructureresources | 否 | 否 |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!IMPORTANT]
> 请参阅[经典部署移动指南](./move-limitations/classic-model-move-limitations.md)。 可以使用特定于该方案的操作跨订阅移动经典部署资源。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | capabilities | 否 | 否 |
> | expressroutecrossconnections | 否 | 否 |
> | expressroutecrossconnections / peerings | 否 | 否 |
> | gatewaysupporteddevices | 否 | 否 |
> | networksecuritygroups | 否 | 否 |
> | quotas | 否 | 否 |
> | reservedips | 否 | 否 |
> | virtualnetworks | 否 | 否 |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!IMPORTANT]
> 请参阅[经典部署移动指南](./move-limitations/classic-model-move-limitations.md)。 可以使用特定于该方案的操作跨订阅移动经典部署资源。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | disks | 否 | 否 |
> | images | 否 | 否 |
> | osimages | 否 | 否 |
> | osplatformimages | 否 | 否 |
> | publicimages | 否 | 否 |
> | quotas | 否 | 否 |
> | storageaccounts | 是 | 否 |
> | vmimages | 否 | 否 |

## <a name="microsoftclassicsubscription"></a>Microsoft.ClassicSubscription

> [!IMPORTANT]
> 请参阅[经典部署移动指南](./move-limitations/classic-model-move-limitations.md)。 可以使用特定于该方案的操作跨订阅移动经典部署资源。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | 操作 | 否 | 否 |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | accounts | 是 | 是 |

<!--Not Available on ## Microsoft.Commerce-->

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!IMPORTANT]
> 请参阅[虚拟机移动指南](./move-limitations/virtual-machines-move-limitations.md)。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | availabilitysets | 是 | 是 |
> | diskaccesses | 否 | 否 |
> | diskencryptionsets | 否 | 否 |
> | disks | 是 | 是 |
> | galleries | 否 | 否 |
> | galleries/images | 否 | 否 |
> | galleries/images/versions | 否 | 否 |
> | hostgroups | 否 | 否 |
> | hostgroups/hosts | 否 | 否 |
> | images | 是 | 是 |
> | proximityplacementgroups | 是 | 是 |
> | restorepointcollections | 否 | 否 |
> | restorepointcollections / restorepoints | 否 | 否 |
> | sharedvmextensions | 否 | 否 |
> | sharedvmimages | 否 | 否 |
> | sharedvmimages/versions | 否 | 否 |
> | snapshots | 是 | 是 |
> | sshpublickeys | 否 | 否 |
> | virtualmachines | 是 | 是 |
> | virtualmachines/extensions | 是 | 是 |
> | virtualmachinescalesets | 是 | 是 |

<!-- Not Available on ## Microsoft.Consumption-->

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | containergroups | 否 | 否 |
> | serviceassociationlinks | 否 | 否 |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | registries | 是 | 是 |
> | registries / agentpools | 是 | 是 |
> | registries/buildtasks | 是 | 是 |
> | registries/replications | 是 | 是 |
> | registries/tasks | 是 | 是 |
> | registries/webhooks | 是 | 是 |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | containerservices | 否 | 否 |
> | managedclusters | 否 | 否 |
> | openshiftmanagedclusters | 否 | 否 |

<!-- Not Available on ## Microsoft.ContentModerator-->
<!-- Not Available on ## Microsoft.CortanaAnalytics-->
<!-- Not Available on ## Microsoft.CostManagement-->
<!-- Not Available on ## Microsoft.CustomerInsights-->
<!-- Not Available on ## Microsoft.CustomerLockbox-->
<!-- Not Available on ## Microsoft.CustomProviders-->

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | jobs | 否 | 否 |

<!-- Not Available on ## Microsoft.DataBoxEdge-->
<!-- Not Available on ## Microsoft.Databricks-->
<!-- Not Available on ## Microsoft.DataCatalog-->
<!-- Not Available on## Microsoft.DataConnect-->
<!-- Not Available on## Microsoft.DataExchange-->

<a name="microsoftdatafactory"></a>
## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | datafactories | 是 | 是 |
> | datafactoryschema | 否 | 否 |
> | factories | 是 | 是 |

<!-- Not Available on ## Microsoft.DataLake-->
<!-- Not Available on ## Microsoft.DataLakeAnalytics-->
<!-- Not Available on ## Microsoft.DataLakeStore-->

<a name="microsoftdatamigration"></a>
## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | services | 否 | 否 |
> | services/projects | 否 | 否 |
> | slots | 否 | 否 |

<!-- Not Available on ## Microsoft.DataProtection-->
<!-- Not Available on ## Microsoft.DataShare-->

<a name="microsoftdbformariadb"></a>
## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | servers | 是 | 是 |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | flexibleServers | 是 | 是 |
> | servers | 是 | 是 |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | flexibleServers | 是 | 是 |
> | servergroups | 否 | 否 |
> | servers | 是 | 是 |
> | serversv2 | 是 | 是 |
> | singleservers | 是 | 是 |

<!--Not Available on ## Microsoft.DeploymentManager-->
<!--Not Available on ## Microsoft.DesktopVirtualization-->

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | elasticpools | 否 | 否 |
> | elasticpools/iothubtenants | 否 | 否 |
> | iothubs | 是 | 是 |
> | provisioningservices | 是 | 是 |

<!--Not Available on ## Microsoft.DevOps-->
<!-- Not Available on ## Microsoft.DevSpaces-->
<!-- Not Available on ## Microsoft.DevTestLab-->
<!--Not Available on ## Microsoft.DigitalTwins-->

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | databaseaccountnames | 否 | 否 |
> | databaseaccounts | 是 | 是 |

<!-- Not Available on ## Microsoft.DomainRegistration-->
<!-- Not Available on ## Microsoft.EnterpriseKnowledgeGraph-->

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | domains | 是 | 是 |
> | eventsubscriptions | 否 - 无法独立移动，但会自动随订阅的资源一起移动。 | 否 - 无法独立移动，但会自动随订阅的资源一起移动。 |
> | extensiontopics | 否 | 否 |
> | partnernamespaces | 是 | 是 |
> | partnerregistrations | 否 | 否 |
> | partnertopics | 是 | 是 |
> | systemtopics | 是 | 是 |
> | topics | 是 | 是 |
> | topictypes | 否 | 否 |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | clusters | 是 | 是 |
> | namespaces | 是 | 是 |
> | sku | 否 | 否 |

<!-- Not Available on ## Microsoft.Experimentation-->
<!-- Not Available on ## Microsoft.Falcon-->

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | featureproviders | 否 | 否 |
> | features | 否 | 否 |
> | providers | 否 | 否 |
> | subscriptionfeatureregistrations | 否 | 否 |

<!-- Not Available on ## Microsoft.Genomics-->

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | automanagedaccounts | 否 | 否 |
> | automanagedvmconfigurationprofiles | 否 | 否 |
> | guestconfigurationassignments | 否 | 否 |
> | software | 否 | 否 |
> | softwareupdateprofile | 否 | 否 |
> | softwareupdates | 否 | 否 |

<!-- Not Available on ## Microsoft.HanaOnAzure-->
<!--Not Available on ## Microsoft.HardwareSecurityModules-->

<a name="microsofthdinsight"></a>
## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!IMPORTANT]
> 可以将 HDInsight 群集移到新订阅或资源组。 但是，无法在订阅之间移动链接到 HDInsight 群集的网络资源（例如虚拟网络、NIC 或负载均衡器）。 此外，无法将连接到群集的虚拟机的 NIC 移到新的资源组。
>
> 将 HDInsight 群集移至新订阅时，请先移动其他资源（例如存储帐户）。 然后移动 HDInsight 群集本身。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | clusters | 是 | 是 |

<!-- Not Available on ## Microsoft.HealthcareApis-->
<!-- Not Available on ## Microsoft.HybridCompute-->
<!-- Not Available on ## Microsoft.HybridData-->
<!-- Not Available on ## HybridNetwork-->
<!--Not Available on ## Microsoft.Hydra-->

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | jobs | 是 | 是 |

## <a name="microsoftinsights"></a>microsoft.insights

> [!IMPORTANT]
> 确保移动到新订阅时，不会超出[订阅配额](azure-subscription-service-limits.md#azure-monitor-limits)。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | actiongroups | 是 | 是 |
> | activitylogalerts | 否 | 否 |
> | alertrules | 是 | 是 |
> | autoscalesettings | 是 | 是 |
> | baseline | 否 | 否 |
> | components | 是 | 是 |
> | datacollectionrules | 否 | 否 |
> | diagnosticsettings | 否 | 否 |
> | diagnosticsettingscategories | 否 | 否 |
> | eventcategories | 否 | 否 |
> | eventtypes | 否 | 否 |
> | extendeddiagnosticsettings | 否 | 否 |
> | guestdiagnosticsettings | 否 | 否 |
> | listmigrationdate | 否 | 否 |
> | logdefinitions | 否 | 否 |
> | logprofiles | 否 | 否 |
> | 日志 | 否 | 否 |
> | metricalerts | 否 | 否 |
> | metricbaselines | 否 | 否 |
> | metricbatch | 否 | 否 |
> | metricdefinitions | 否 | 否 |
> | metricnamespaces | 否 | 否 |
> | 指标 | 否 | 否 |
> | migratealertrules | 否 | 否 |
> | migratetonewpricingmodel | 否 | 否 |
> | myworkbooks | 否 | 否 |
> | notificationgroups | 否 | 否 |
> | privatelinkscopes | 否 | 否 |
> | rollbacktolegacypricingmodel | 否 | 否 |
> | scheduledqueryrules | 是 | 是 |
> | 拓扑 | 否 | 否 |
> | 事务 | 否 | 否 |
> | vminsightsonboardingstatuses | 否 | 否 |
> | webtests | 是 | 是 |
> | webtests/gettestresultfile | 否 | 否 |
> | workbooks | 是 | 是 |
> | workbooktemplates | 是 | 是 |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | apptemplates | 否 | 否 |
> | iotapps | 是 | 是 |

<!-- Not Available on ## Microsoft.IoTSpaces-->

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!IMPORTANT]
> 用于磁盘加密的 Key Vault 不能移到同一订阅中的资源组，也不能跨订阅移动。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | deletedvaults | 否 | 否 |
> | hsmpools | 否 | 否 |
> | managedhsms | 否 | 否 |
> | vaults | 是 | 是 |

<!--Not Available on ## Microsoft.Kubernetes-->
<!-- Not Available on ## Microsoft.Kubernetes-->

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | clusters | 是 | 是 |


<!-- Not Available on ## Microsoft.LabServices-->
<!-- Not Available on ## Microsoft.LocationBasedServices-->
<!-- Not Available on ## Microsoft.LocationServices-->

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | hostingenvironments | 否 | 否 |
> | integrationaccounts | 是 | 是 |
> | integrationserviceenvironments | 是 | 否 |
> | integrationserviceenvironments/managedapis | 是 | 否 |
> | isolatedenvironments | 否 | 否 |
> | workflows | 是 | 是 |

<!-- Not Available on ## Microsoft.MachineLearning-->
<!-- Not Available on ## Microsoft.MachineLearningCompute-->
<!-- Not Available on ## Microsoft.MachineLearningExperimentation-->
<!-- Not Available on ## Microsoft.MachineLearningModelManagement-->

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | workspaces | 否 | 否 |

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | configurationassignments | 否 | 否 |
> | maintenanceconfigurations | 是 | 是 |
> | updates | 否 | 否 |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | identities | 否 | 否 |
> | userassignedidentities | 否 | 否 |

<!-- Not Available on ## Microsoft.ManagedNetwork-->

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | marketplaceregistrationdefinitions | 否 | 否 |
> | registrationassignments | 否 | 否 |
> | registrationdefinitions | 否 | 否 |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | getentities | 否 | 否 |
> | managementgroups | 否 | 否 |
> | managementgroups / settings | 否 | 否 |
> | resources | 否 | 否 |
> | starttenantbackfill | 否 | 否 |
> | tenantbackfillstatus | 否 | 否 |

<!-- Not Available on ## Microsoft.Maps-->
<!--Not Available on ## Microsoft.Marketplace-->
<!-- Not Available on ## Microsoft.MarketplaceApps-->
<!--Not Available on ## Microsoft.MarketplaceOrdering-->

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | mediaservices | 是 | 是 |
> | mediaservices/liveevents | 是 | 是 |
> | mediaservices/streamingendpoints | 是 | 是 |

<!-- Not Available on ## Microsoft.Microservices4Spring-->
<!-- Not Available on ## Microsoft.Migrate-->
<!--Not Available on ## Microsoft.MixedReality-->
<!-- Not Available on ## Microsoft.NetApp-->
<!--DDos, and Front Door are not available on Mooncake-->
<!--MOONCAKE CUSTOMIZEIONT-->

## <a name="microsoftnetwork"></a>Microsoft.Network

> [!IMPORTANT]
> 请参阅[网络移动指南](./move-limitations/networking-move-limitations.md)。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | applicationgateways | 否 | 否 |
> | applicationgatewaywebapplicationfirewallpolicies | 否 | 否 |
> | applicationsecuritygroups | 是 | 是 |
> | azurefirewalls | 否 | 否 |
> | bastionhosts | 否 | 否 |
> | bgpservicecommunities | 否 | 否 |
> | connections | 是 | 是 |
> | dnszones | 是 | 是 |
> | expressroutecircuits | 否 | 否 |
> | expressroutegateways | 否 | 否 |
> | expressrouteserviceproviders | 否 | 否 |
> | firewallpolicies | 是 | 是 |
> | ipallocations | 是 | 是 |
> | ipgroups | 是 | 是 |
> | loadbalancers | 是 - 基本 SKU<br /> 是 - 标准 SKU | 是 - 基本 SKU<br />否 - 标准 SKU |
> | localnetworkgateways | 是 | 是 |
> | natgateways | 否 | 否 |
> | networkexperimentprofiles | 否 | 否 |
> | networkintentpolicies | 是 | 是 |
> | networkinterfaces | 是 | 是 |
> | networkprofiles | 否 | 否 |
> | networksecuritygroups | 是 | 是 |
> | networkwatchers | 是 | 否 |
> | networkwatchers/connectionmonitors | 是 | 否 |
> | networkwatchers/flowlogs | 是 | 否 |
> | networkwatchers/pingmeshes | 是 | 否 |
> | p2svpngateways | 否 | 否 |
> | privatednszones | 是 | 是 |
> | privatednszones/virtualnetworklinks | 是 | 是 |
> | privatednszonesinternal | 否 | 否 |
> | privateendpointredirectmaps | 否 | 否 |
> | privateendpoints | 是 | 是 |
> | privatelinkservices | 否 | 否 |
> | publicipaddresses | 是 - 基本 SKU<br />是 - 标准 SKU | 是 - 基本 SKU<br />否 - 标准 SKU |
> | publicipprefixes | 是 | 是 |
> | routefilters | 否 | 否 |
> | routetables | 是 | 是 |
> | securitypartnerproviders | 是 | 是 |
> | serviceendpointpolicies | 是 | 是 |
> | trafficmanagergeographichierarchies | 否 | 否 |
> | trafficmanagerprofiles | 是 | 是 |
> | trafficmanagerprofiles / heatmaps | 否 | 否 |
> | trafficmanagerusermetricskeys | 否 | 否 |
> | virtualhubs | 否 | 否 |
> | virtualnetworkgateways | 是 | 是 |
> | virtualnetworks | 是 | 是 |
> | virtualnetworktaps | 否 | 否 |
> | virtualrouters | 是 | 是 |
> | virtualwans | 否 | 否 |
> | vpngateways（虚拟 WAN） | 否 | 否 |
> | vpnserverconfigurations | 否 | 否 |
> | vpnsites（虚拟 WAN） | 否 | 否 |

<!--DDos, and Front Door are not available on Mooncake-->
<!--MOONCAKE CUSTOMIZEIONT-->

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | namespaces | 是 | 是 |
> | namespaces/notificationhubs | 是 | 是 |

## <a name="microsoftobjectstore"></a>Microsoft.ObjectStore

<!-- Not Available on ## Microsoft.ObjectStore-->
<--在 ## Microsoft.OffAzure 上不可用-->

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!IMPORTANT]
> 确保移动到新订阅时，不会超出[订阅配额](azure-subscription-service-limits.md#azure-monitor-limits)。
>
> 无法移动具有链接的自动化帐户的工作区。 在开始移动操作之前，请确保取消链接所有自动化帐户。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | clusters | 否 | 否 |
> | deletedworkspaces | 否 | 否 |
> | linktargets | 否 | 否 |
> | storageinsightconfigs | 否 | 否 |
> | workspaces | 是 | 是 |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | managementassociations | 否 | 否 |
> | managementconfigurations | 是 | 是 |
> | solutions | 是 | 是 |
> | 视图 | 是 | 是 |

<!--Not Available on ## Microsoft.Peering-->

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | policyevents | 否 | 否 |
> | policystates | 否 | 否 |
> | policytrackedresources | 否 | 否 |
> | remediations | 否 | 否 |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | consoles | 否 | 否 |
> | dashboards | 是 | 是 |
> | usersettings | 否 | 否 |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | workspacecollections | 是 | 是 |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | capacities | 是 | 是 |

<!--Not Available on ## Microsoft.ProjectBabylon-->
<!--Not Available on ## Microsoft.ProviderHub-->
<!--Not Available on ## Microsoft.Quantum-->

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

<!--Not Available on [Recovery Services move guidance](../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure-resource-manager/toc.json)-->

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | replicationeligibilityresults | 否 | 否 |
> | vaults | 是 | 是 |

<!--Not Available on ## Microsoft.RedHatOpenShift-->

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | namespaces | 是 | 是 |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | 查询 | 是 | 是 |
> | resourcechangedetails | 否 | 否 |
> | resourcechanges | 否 | 否 |
> | resources | 否 | 否 |
> | resourceshistory | 否 | 否 |
> | subscriptionsstatus | 否 | 否 |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | childresources | 否 | 否 |
> | emergingissues | 否 | 否 |
> | events | 否 | 否 |
> | metadata | 否 | 否 |
> | 通知 | 否 | 否 |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | deployments | 否 | 否 |
> | deploymentscripts | 否 | 否 |
> | deploymentscripts / logs | 否 | 否 |
> | 链接 | 否 | 否 |
> | providers | 否 | 否 |
> | resourcegroups | 否 | 否 |
> | resources | 否 | 否 |
> | subscriptions | 否 | 否 |
> | 标记 | 否 | 否 |
> | templatespecs | 否 | 否 |
> | templatespecs / versions | 否 | 否 |
> | tenants | 否 | 否 |

<!--Not Available on ## Microsoft.SaaS-->

## <a name="microsoftsearch"></a>Microsoft.Search

> [!IMPORTANT]
> 不能通过一项操作移动不同区域中的多个搜索资源。 只能通过多个单独的操作移动它们。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | resourcehealthmetadata | 否 | 否 |
> | searchservices | 是 | 是 |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | adaptivenetworkhardenings | 否 | 否 |
> | advancedthreatprotectionsettings | 否 | 否 |
> | alerts | 否 | 否 |
> | allowedconnections | 否 | 否 |
> | applicationwhitelistings | 否 | 否 |
> | assessmentmetadata | 否 | 否 |
> | assessments | 否 | 否 |
> | autodismissalertsrules | 否 | 否 |
> | automations | 是 | 是 |
> | autoprovisioningsettings | 否 | 否 |
> | complianceresults | 否 | 否 |
> | compliances | 否 | 否 |
> | datacollectionagents | 否 | 否 |
> | devicesecuritygroups | 否 | 否 |
> | discoveredsecuritysolutions | 否 | 否 |
> | externalsecuritysolutions | 否 | 否 |
> | informationprotectionpolicies | 否 | 否 |
> | iotsecuritysolutions | 是 | 是 |
> | iotsecuritysolutions / analyticsmodels | 否 | 否 |
> | iotsecuritysolutions / analyticsmodels / aggregatedalerts | 否 | 否 |
> | iotsecuritysolutions / analyticsmodels / aggregatedrecommendations | 否 | 否 |
> | jitnetworkaccesspolicies | 否 | 否 |
> | 策略 | 否 | 否 |
> | pricings | 否 | 否 |
> | regulatorycompliancestandards | 否 | 否 |
> | regulatorycompliancestandards / regulatorycompliancecontrols | 否 | 否 |
> | regulatorycompliancestandards / regulatorycompliancecontrols / regulatorycomplianceassessments | 否 | 否 |
> | securitycontacts | 否 | 否 |
> | securitysolutions | 否 | 否 |
> | securitysolutionsreferencedata | 否 | 否 |
> | securitystatuses | 否 | 否 |
> | securitystatusessummaries | 否 | 否 |
> | servervulnerabilityassessments | 否 | 否 |
> | 设置 | 否 | 否 |
> | subassessments | 否 | 否 |
> | 任务 | 否 | 否 |
> | topologies | 否 | 否 |
> | workspacesettings | 否 | 否 |

<!-- Not Available on ## Microsoft.SecurityInsights-->
<!-- Not Available on ## Microsoft.SerialConsole-->
<!-- Not Available on ## Microsoft.ServerManagement-->

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | namespaces | 是 | 是 |
> | premiummessagingregions | 否 | 否 |
> | sku | 否 | 否 |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | applications | 否 | 否 |
> | clusters | 是 | 是 |
> | containergroups | 否 | 否 |
> | containergroupsets | 否 | 否 |
> | edgeclusters | 否 | 否 |
> | managedclusters | 否 | 否 |
> | networks | 否 | 否 |
> | secretstores | 否 | 否 |
> | volumes | 否 | 否 |

<!-- Not Available on ## Microsoft.ServiceFabricMesh-->
<!-- Not Available on ## Microsoft.Services-->

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | signalr | 是 | 是 |

<!-- Not Available on ## Microsoft.SoftwarePlan-->

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | applicationdefinitions | 否 | 否 |
> | applications | 否 | 否 |
> | jitrequests | 否 | 否 |

## <a name="microsoftsql"></a>Microsoft.Sql

> [!IMPORTANT]
> 数据库和服务器必须位于同一个资源组中。 移动 SQL 服务器时，也会移动其所有数据库。 此行为适用于 Azure SQL 数据库和 Azure Synapse Analytics 数据库。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | instancepools | 否 | 否 |
> | locations | 是 | 是 |
> | managedinstances | 否 | 否 |
> | servers | 是 | 是 |
> | servers/databases | 是 | 是 |
> | servers / databases / backuplongtermretentionpolicies | 是 | 是 |
> | servers/elasticpools | 是 | 是 |
> | servers/jobaccounts | 是 | 是 |
> | servers/jobagents | 是 | 是 |
> | virtualclusters | 是 | 是 |

<!-- Not Available on ## Microsoft.SqlVirtualMachine-->

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | storageaccounts | 是 | 是 |

<!-- Not Available on ## Microsoft.StorageCache-->
<!-- Not Available on ## Microsoft.StorageSync-->
<!-- Not Available on ## Microsoft.StorageSyncDev-->
<!-- Not Available on ## Microsoft.StorageSyncInt-->
<!-- Not Available on ## Microsoft.StorSimple-->

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!IMPORTANT]
> 当流分析作业处于运行状态时，则无法进行移动。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | clusters | 否 | 否 |
> | streamingjobs | 是 | 是 |

<!-- Not Available on ## Microsoft.StreamAnalyticsExplorer-->
<!-- Not Available on ## Microsoft.Subscription-->
<!-- Not Available on ## microsoft.support-->
<!-- Not Available on ## Microsoft.Synapse->


<a name="microsofttimeseriesinsights"></a>

## Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Resource type | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | environments | Yes | Yes |
> | environments / eventsources | Yes | Yes |
> | environments / referencedatasets | Yes | Yes |

<!-- Not Available on ## Microsoft.Token-->
<!-- Not Available on ## Microsoft.VirtualMachineImages-->
<!-- Not Available on ## microsoft.visualstudio-->
<!-- Not Available on ## Microsoft.VMware-->
<!-- Not Available on ## Microsoft.VMwareCloudSimple-->
<!-- Not Available on ## Microsoft.VnfManager-->
<!-- Not Available on ## Microsoft.VSOnline-->

## <a name="microsoftweb"></a>Microsoft.Web

> [!IMPORTANT]
> 请参阅[应用服务移动指南](./move-limitations/app-service-move-limitations.md)。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | availablestacks | 否 | 否 |
> | billingmeters | 否 | 否 |
> | certificates | 否 | 是 |
> | connectiongateways | 是 | 是 |
> | connections | 是 | 是 |
> | customapis | 是 | 是 |
> | deletedsites | 否 | 否 |
> | deploymentlocations | 否 | 否 |
> | georegions | 否 | 否 |
> | hostingenvironments | 否 | 否 |
> | kubeenvironments | 是 | 是 |
> | publishingusers | 否 | 否 |
> | 建议 | 否 | 否 |
> | resourcehealthmetadata | 否 | 否 |
> | runtimes | 否 | 否 |
> | serverfarms | 是 | 是 |
> | serverfarms / eventgridfilters | 否 | 否 |
> | sites | 是 | 是 |
> | sites/premieraddons | 是 | 是 |
> | sites/slots | 是 | 是 |
> | sourcecontrols | 否 | 否 |
> | staticsites | 否 | 否 |

<!-- Not Available on ## Microsoft.WindowsESU-->
<!-- Not Available on ## Microsoft.WindowsIoT-->
<!-- Not Available on## Microsoft.WorkloadBuilder-->
<!-- Not Available on ## Microsoft.WorkloadMonitor-->

## <a name="third-party-services"></a>第三方服务

第三方服务目前不支持移动操作。

## <a name="next-steps"></a>后续步骤

有关用于移动资源的命令，请参阅[将资源移到新资源组或订阅](move-resource-group-and-subscription.md)。

<!--Not Avaialble on [move-support-resources.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/move-support-resources.csv)-->
<!-- Update_Description: update meta properties, wording update, update link -->