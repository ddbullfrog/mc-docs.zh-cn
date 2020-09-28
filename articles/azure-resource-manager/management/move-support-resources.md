---
title: 资源类型支持的移动操作
description: 列出可移到新资源组或订阅的 Azure 资源类型。
ms.topic: conceptual
origin.date: 09/08/2020
author: rockboyfor
ms.date: 09/21/2020
ms.testscope: yes
ms.testdate: 08/24/2020
ms.author: v-yeche
ms.openlocfilehash: 12b1d364d5cea853b3f741c92b4152a2f3c9c217
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146413"
---
<!--Verify Successfully-->
# <a name="move-operation-support-for-resources"></a>支持移动操作的资源

本文列出某个 Azure 资源类型是否支持移动操作。 它还提供了有关移动资源时要考虑的特殊条件的信息。

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
> | domainservices / oucontainer | 否 | 否 |
> | locations | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | 操作 | 否 | 否 |

## <a name="microsoftaadiam"></a>microsoft.aadiam

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | diagnosticsettings | 否 | 否 |
> | diagnosticsettingscategories | 否 | 否 |
> | 操作 | 否 | 否 |
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
> | 操作 | 否 | 否 |
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
> | 操作 | 否 | 否 |
> | smartdetectoralertrules | 是 | 是 |
> | smartgroups | 否 | 否 |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | locations | 否 | 否 |
> | locations / checknameavailability | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / operationstatuses | 否 | 否 |
> | 操作 | 否 | 否 |
> | servers | 是 | 是 |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!IMPORTANT]
> 无法移动设置为消耗 SKU 的 API 管理服务。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | checkfeedbackrequired | 否 | 否 |
> | checknameavailability | 否 | 否 |
> | checkservicenameavailability | 否 | 否 |
> | 操作 | 否 | 否 |
> | reportfeedback | 否 | 否 |
> | 服务 | 是 | 是 |
> | validateservicename | 否 | 否 |

<!--Not Available on ## Microsoft.AppConfiguration-->
<!--Not Available on ## Microsoft.AppPlatform-->
<!--Not Available on ## Microsoft.AppService-->
<!--Not Available on ## Microsoft.Attestation-->

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | checkaccess | 否 | 否 |
> | classicadministrators | 否 | 否 |
> | dataaliases | 否 | 否 |
> | denyassignments | 否 | 否 |
> | elevateaccess | 否 | 否 |
> | findorphanroleassignments | 否 | 否 |
> | 锁定 | 否 | 否 |
> | 操作 | 否 | 否 |
> | operationstatus | 否 | 否 |
> | 权限 | 否 | 否 |
> | policyassignments | 否 | 否 |
> | policydefinitions | 否 | 否 |
> | policysetdefinitions | 否 | 否 |
> | privatelinkassociations | 否 | 否 |
> | provideroperations | 否 | 否 |
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
> | automationaccounts / jobs | 否 | 否 |
> | automationaccounts / privateendpointconnectionproxies | 否 | 否 |
> | automationaccounts / privateendpointconnections | 否 | 否 |
> | automationaccounts / privatelinkresources | 否 | 否 |
> | automationaccounts / runbooks | 是 | 是 |
> | automationaccounts / softwareupdateconfigurations | 否 | 否 |
> | automationaccounts / webhooks | 否 | 否 |
> | 操作 | 否 | 否 |

<!--Not Available on ## Microsoft.AVS-->

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | b2cdirectories | 是 | 是 |
> | b2ctenants | 否 | 否 |
> | checknameavailability | 否 | 否 |
> | 操作 | 否 | 否 |

<!-- Not Available on ## Microsoft.AzureData-->

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | cloudmanifestfiles | 否 | 否 |
> | 操作 | 否 | 否 |
> | registrations | 是 | 是 |
> | registrations / customersubscriptions | 否 | 否 |
> | registrations / products | 否 | 否 |

<!--Not Available on ## Microsoft.AzureStackHCI-->

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | batchaccounts | 是 | 是 |
> | locations | 否 | 否 |
> | locations / accountoperationresults | 否 | 否 |
> | locations / checknameavailability | 否 | 否 |
> | locations / quotas | 否 | 否 |
> | 操作 | 否 | 否 |

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
> | blueprintassignments / assignmentoperations | 否 | 否 |
> | blueprintassignments / operations | 否 | 否 |
> | blueprints | 否 | 否 |
> | blueprints / artifacts | 否 | 否 |
> | blueprints / versions | 否 | 否 |
> | blueprints / versions / artifacts | 否 | 否 |
> | 操作 | 否 | 否 |

<!-- Not Available on ## Microsoft.BotService-->

## <a name="microsoftcache"></a>Microsoft.Cache

> [!IMPORTANT]
> 如果 Azure Redis 缓存实例配置了虚拟网络，则实例无法被移动到其他订阅。 请参阅[网络移动限制](./move-limitations/networking-move-limitations.md)。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | checknameavailability | 否 | 否 |
> | locations | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / operationsstatus | 否 | 否 |
> | 操作 | 否 | 否 |
> | redis | 是 | 是 |
> | redis/eventgridfilters | 否 | 否 |
> | redis / privatelinkresources | 否 | 否 |
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
> | checknameavailability | 否 | 否 |
> | checkresourceusage | 否 | 否 |
> | edgenodes | 否 | 否 |
> | operationresults | 否 | 否 |
> | operationresults / profileresults | 否 | 否 |
> | operationresults / profileresults / endpointresults | 否 | 否 |
> | operationresults / profileresults / endpointresults / customdomainresults | 否 | 否 |
> | operationresults / profileresults / endpointresults / origingroupresults | 否 | 否 |
> | operationresults / profileresults / endpointresults / originresults | 否 | 否 |
> | 操作 | 否 | 否 |
> | 配置文件 | 是 | 否 |
> | profiles/endpoints | 是 | 否 |
> | profiles / endpoints / customdomains | 否 | 否 |
> | profiles / endpoints / origingroups | 否 | 否 |
> | profiles / endpoints / origins | 否 | 否 |
> | validateprobe | 否 | 否 |

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
> | checkdomainnameavailability | 否 | 否 |
> | domainnames | 是 | 否 |
> | domainnames / capabilities | 否 | 否 |
> | domainnames / internalloadbalancers | 否 | 否 |
> | domainnames / servicecertificates | 否 | 否 |
> | domainnames / slots | 否 | 否 |
> | domainnames / slots / roles | 否 | 否 |
> | domainnames / slots / roles / metricdefinitions | 否 | 否 |
> | domainnames / slots / roles / metrics | 否 | 否 |
> | movesubscriptionresources | 否 | 否 |
> | operatingsystemfamilies | 否 | 否 |
> | operatingsystems | 否 | 否 |
> | 操作 | 否 | 否 |
> | operationstatuses | 否 | 否 |
> | quotas | 否 | 否 |
> | resourcetypes | 否 | 否 |
> | validatesubscriptionmoveavailability | 否 | 否 |
> | virtualmachines | 是 | 是 |
> | virtualmachines / diagnosticsettings | 否 | 否 |
> | virtualmachines / metricdefinitions | 否 | 否 |
> | virtualmachines / metrics | 否 | 否 |

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
> | 操作 | 否 | 否 |
> | quotas | 否 | 否 |
> | reservedips | 否 | 否 |
> | virtualnetworks | 否 | 否 |
> | virtualnetworks / remotevirtualnetworkpeeringproxies | 否 | 否 |
> | virtualnetworks / virtualnetworkpeerings | 否 | 否 |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!IMPORTANT]
> 请参阅[经典部署移动指南](./move-limitations/classic-model-move-limitations.md)。 可以使用特定于该方案的操作跨订阅移动经典部署资源。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | capabilities | 否 | 否 |
> | checkstorageaccountavailability | 否 | 否 |
> | disks | 否 | 否 |
> | images | 否 | 否 |
> | 操作 | 否 | 否 |
> | osimages | 否 | 否 |
> | osplatformimages | 否 | 否 |
> | publicimages | 否 | 否 |
> | quotas | 否 | 否 |
> | storageaccounts | 是 | 否 |
> | storageaccounts / blobservices | 否 | 否 |
> | storageaccounts / fileservices | 否 | 否 |
> | storageaccounts / metricdefinitions | 否 | 否 |
> | storageaccounts / metrics | 否 | 否 |
> | storageaccounts / queueservices | 否 | 否 |
> | storageaccounts / services | 否 | 否 |
> | storageaccounts / services / diagnosticsettings | 否 | 否 |
> | storageaccounts / services / metricdefinitions | 否 | 否 |
> | storageaccounts / services / metrics | 否 | 否 |
> | storageaccounts / tableservices | 否 | 否 |
> | storageaccounts / vmimages | 否 | 否 |
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
> | checkdomainavailability | 否 | 否 |
> | locations | 否 | 否 |
> | locations / checkskuavailability | 否 | 否 |
> | locations / deletevirtualnetworkorsubnets | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | 操作 | 否 | 否 |

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
> | locations | 否 | 否 |
> | locations / artifactpublishers | 否 | 否 |
> | locations / capsoperations | 否 | 否 |
> | locations / diskoperations | 否 | 否 |
> | locations / loganalytics | 否 | 否 |
> | locations / operations | 否 | 否 |
> | locations / publishers | 否 | 否 |
> | locations / runcommands | 否 | 否 |
> | locations / usages | 否 | 否 |
> | locations / virtualmachines | 否 | 否 |
> | locations / vmsizes | 否 | 否 |
> | locations / vsmoperations | 否 | 否 |
> | 操作 | 否 | 否 |
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
> | virtualmachines / metricdefinitions | 否 | 否 |
> | virtualmachines / runcommands | 否 | 否 |
> | virtualmachinescalesets | 是 | 是 |
> | virtualmachinescalesets / extensions | 否 | 否 |
> | virtualmachinescalesets / networkinterfaces | 否 | 否 |
> | virtualmachinescalesets / publicipaddresses | 否 | 否 |
> | virtualmachinescalesets / virtualmachines | 否 | 否 |
> | virtualmachinescalesets / virtualmachines / networkinterfaces | 否 | 否 |

<!-- Not Available on ## Microsoft.Consumption-->

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | containergroups | 否 | 否 |
> | locations | 否 | 否 |
> | locations / cachedimages | 否 | 否 |
> | locations / capabilities | 否 | 否 |
> | locations / deletevirtualnetworkorsubnets | 否 | 否 |
> | locations / operations | 否 | 否 |
> | locations / usages | 否 | 否 |
> | 操作 | 否 | 否 |
> | serviceassociationlinks | 否 | 否 |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | checknameavailability | 否 | 否 |
> | locations | 否 | 否 |
> | locations / authorize | 否 | 否 |
> | locations / deletevirtualnetworkorsubnets | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / setupauth | 否 | 否 |
> | 操作 | 否 | 否 |
> | registries | 是 | 是 |
> | registries / agentpools | 是 | 是 |
> | registries / agentpools / listqueuestatus | 否 | 否 |
> | registries / builds | 否 | 否 |
> | registries / builds / cancel | 否 | 否 |
> | registries / builds / getloglink | 否 | 否 |
> | registries / buildtasks | 是 | 是 |
> | registries / buildtasks / listsourcerepositoryproperties | 否 | 否 |
> | registries / buildtasks / steps | 否 | 否 |
> | registries / buildtasks / steps / listbuildarguments | 否 | 否 |
> | registries / eventgridfilters | 否 | 否 |
> | registries / exportpipelines | 否 | 否 |
> | registries / generatecredentials | 否 | 否 |
> | registries / getbuildsourceuploadurl | 否 | 否 |
> | registries / getcredentials | 否 | 否 |
> | registries / importimage | 否 | 否 |
> | registries / importpipelines | 否 | 否 |
> | registries / listbuildsourceuploadurl | 否 | 否 |
> | registries / listcredentials | 否 | 否 |
> | registries / listpolicies | 否 | 否 |
> | registries / listusages | 否 | 否 |
> | registries / pipelineruns | 否 | 否 |
> | registries / privateendpointconnectionproxies | 否 | 否 |
> | registries / privateendpointconnectionproxies / validate | 否 | 否 |
> | registries / privateendpointconnections | 否 | 否 |
> | registries / privatelinkresources | 否 | 否 |
> | registries / queuebuild | 否 | 否 |
> | registries / regeneratecredential | 否 | 否 |
> | registries / regeneratecredentials | 否 | 否 |
> | registries / replications | 是 | 是 |
> | registries / runs | 否 | 否 |
> | registries / runs / cancel | 否 | 否 |
> | registries / runs / listlogsasurl | 否 | 否 |
> | registries / schedulerun | 否 | 否 |
> | registries / scopemaps | 否 | 否 |
> | registries / taskruns | 否 | 否 |
> | registries / taskruns / listdetails | 否 | 否 |
> | registries / tasks | 是 | 是 |
> | registries / tasks / listdetails | 否 | 否 |
> | registries / tokens | 否 | 否 |
> | registries / updatepolicies | 否 | 否 |
> | registries / webhooks | 是 | 是 |
> | registries / webhooks / getcallbackconfig | 否 | 否 |
> | registries / webhooks / listevents | 否 | 否 |
> | registries / webhooks / ping | 否 | 否 |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | containerservices | 否 | 否 |
> | locations | 否 | 否 |
> | locations / openshiftclusters | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / operations | 否 | 否 |
> | locations / orchestrators | 否 | 否 |
> | managedclusters | 否 | 否 |
> | openshiftmanagedclusters | 否 | 否 |
> | 操作 | 否 | 否 |

<!-- Not Available on ## Microsoft.ContentModerator-->
<!-- Not Available on ## Microsoft.CortanaAnalytics-->
<!-- Not Available on ## Microsoft.CostManagement-->
<!-- Not Available on ## Microsoft.CostManagementExports-->
<!-- Not Available on ## Microsoft.CustomerInsights-->
<!-- Not Available on ## Microsoft.CustomerLockbox-->
<!-- Not Available on ## Microsoft.CustomProviders-->

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | jobs | 否 | 否 |
> | locations | 否 | 否 |
> | locations / availableskus | 否 | 否 |
> | locations / checknameavailability | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / regionconfiguration | 否 | 否 |
> | locations / validateaddress | 否 | 否 |
> | locations / validateinputs | 否 | 否 |
> | 操作 | 否 | 否 |

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
> | checkazuredatafactorynameavailability | 否 | 否 |
> | checkdatafactorynameavailability | 否 | 否 |
> | datafactories | 是 | 是 |
> | datafactories / diagnosticsettings | 否 | 否 |
> | datafactories / metricdefinitions | 否 | 否 |
> | datafactoryschema | 否 | 否 |
> | factories | 是 | 是 |
> | factories / integrationruntimes | 否 | 否 |
> | locations | 否 | 否 |
> | locations / configurefactoryrepo | 否 | 否 |
> | locations / getfeaturevalue | 否 | 否 |
> | 操作 | 否 | 否 |

<!-- Not Available on ## Microsoft.DataLake-->
<!-- Not Available on ## Microsoft.DataLakeAnalytics-->
<!-- Not Available on ## Microsoft.DataLakeStore-->

<a name="microsoftdatamigration"></a>
## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | locations | 否 | 否 |
> | locations / checknameavailability | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / operationstatuses | 否 | 否 |
> | 操作 | 否 | 否 |
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
> | checknameavailability | 否 | 否 |
> | locations | 否 | 否 |
> | locations / azureasyncoperation | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / performancetiers | 否 | 否 |
> | locations / privateendpointconnectionazureasyncoperation | 否 | 否 |
> | locations / privateendpointconnectionoperationresults | 否 | 否 |
> | locations / privateendpointconnectionproxyazureasyncoperation | 否 | 否 |
> | locations / privateendpointconnectionproxyoperationresults | 否 | 否 |
> | locations / recommendedactionsessionsazureasyncoperation | 否 | 否 |
> | locations / recommendedactionsessionsoperationresults | 否 | 否 |
> | locations / securityalertpoliciesazureasyncoperation | 否 | 否 |
> | locations / securityalertpoliciesoperationresults | 否 | 否 |
> | locations / serverkeyazureasyncoperation | 否 | 否 |
> | locations / serverkeyoperationresults | 否 | 否 |
> | 操作 | 否 | 否 |
> | servers | 是 | 是 |
> | servers / advisors | 否 | 否 |
> | servers / privateendpointconnectionproxies | 否 | 否 |
> | servers / privateendpointconnections | 否 | 否 |
> | servers / privatelinkresources | 否 | 否 |
> | servers / querytexts | 否 | 否 |
> | servers / recoverableservers | 否 | 否 |
> | servers / topquerystatistics | 否 | 否 |
> | servers / virtualnetworkrules | 否 | 否 |
> | servers / waitstatistics | 否 | 否 |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | checknameavailability | 否 | 否 |
> | locations | 否 | 否 |
> | locations / administratorazureasyncoperation | 否 | 否 |
> | locations / administratoroperationresults | 否 | 否 |
> | locations / azureasyncoperation | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / performancetiers | 否 | 否 |
> | locations / privateendpointconnectionazureasyncoperation | 否 | 否 |
> | locations / privateendpointconnectionoperationresults | 否 | 否 |
> | locations / privateendpointconnectionproxyazureasyncoperation | 否 | 否 |
> | locations / privateendpointconnectionproxyoperationresults | 否 | 否 |
> | locations / recommendedactionsessionsazureasyncoperation | 否 | 否 |
> | locations / recommendedactionsessionsoperationresults | 否 | 否 |
> | locations / securityalertpoliciesazureasyncoperation | 否 | 否 |
> | locations / securityalertpoliciesoperationresults | 否 | 否 |
> | locations / serverkeyazureasyncoperation | 否 | 否 |
> | locations / serverkeyoperationresults | 否 | 否 |
> | 操作 | 否 | 否 |
> | servers | 是 | 是 |
> | servers / advisors | 否 | 否 |
> | servers / keys | 否 | 否 |
> | servers / privateendpointconnectionproxies | 否 | 否 |
> | servers / privateendpointconnections | 否 | 否 |
> | servers / privatelinkresources | 否 | 否 |
> | servers / querytexts | 否 | 否 |
> | servers / recoverableservers | 否 | 否 |
> | servers / topquerystatistics | 否 | 否 |
> | servers / virtualnetworkrules | 否 | 否 |
> | servers / waitstatistics | 否 | 否 |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | checknameavailability | 否 | 否 |
> | locations | 否 | 否 |
> | locations / administratorazureasyncoperation | 否 | 否 |
> | locations / administratoroperationresults | 否 | 否 |
> | locations / azureasyncoperation | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / performancetiers | 否 | 否 |
> | locations / privateendpointconnectionazureasyncoperation | 否 | 否 |
> | locations / privateendpointconnectionoperationresults | 否 | 否 |
> | locations / privateendpointconnectionproxyazureasyncoperation | 否 | 否 |
> | locations / privateendpointconnectionproxyoperationresults | 否 | 否 |
> | locations / recommendedactionsessionsazureasyncoperation | 否 | 否 |
> | locations / recommendedactionsessionsoperationresults | 否 | 否 |
> | locations / securityalertpoliciesazureasyncoperation | 否 | 否 |
> | locations / securityalertpoliciesoperationresults | 否 | 否 |
> | locations / serverkeyazureasyncoperation | 否 | 否 |
> | locations / serverkeyoperationresults | 否 | 否 |
> | 操作 | 否 | 否 |
> | servergroups | 否 | 否 |
> | servers | 是 | 是 |
> | servers / advisors | 否 | 否 |
> | servers / keys | 否 | 否 |
> | servers / privateendpointconnectionproxies | 否 | 否 |
> | servers / privateendpointconnections | 否 | 否 |
> | servers / privatelinkresources | 否 | 否 |
> | servers / querytexts | 否 | 否 |
> | servers / recoverableservers | 否 | 否 |
> | servers / topquerystatistics | 否 | 否 |
> | servers / virtualnetworkrules | 否 | 否 |
> | servers / waitstatistics | 否 | 否 |
> | serversv2 | 是 | 是 |
> | singleservers | 是 | 是 |

<!--Not Available on ## Microsoft.DeploymentManager-->
<!--Not Available on ## Microsoft.DesktopVirtualization-->

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | checknameavailability | 否 | 否 |
> | checkprovisioningservicenameavailability | 否 | 否 |
> | elasticpools | 否 | 否 |
> | elasticpools/iothubtenants | 否 | 否 |
> | iothubs | 是 | 是 |
> | iothubs / eventgridfilters | 否 | 否 |
> | iothubs/securitysettings | 否 | 否 |
> | operationresults | 否 | 否 |
> | 操作 | 否 | 否 |
> | provisioningservices | 是 | 是 |
> | usages | 否 | 否 |

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
> | locations | 否 | 否 |
> | locations / deletevirtualnetworkorsubnets | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / operationsstatus | 否 | 否 |
> | operationresults | 否 | 否 |
> | 操作 | 否 | 否 |

<!-- Not Available on ## Microsoft.DomainRegistration-->
<!-- Not Available on ## Microsoft.EnterpriseKnowledgeGraph-->

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | domains | 是 | 是 |
> | domains / topics | 否 | 否 |
> | eventsubscriptions | 否 - 无法独立移动，但会自动随订阅的资源一起移动。 | 否 - 无法独立移动，但会自动随订阅的资源一起移动。 |
> | extensiontopics | 否 | 否 |
> | locations | 否 | 否 |
> | locations / eventsubscriptions | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / operationsstatus | 否 | 否 |
> | locations / topictypes | 否 | 否 |
> | operationresults | 否 | 否 |
> | 操作 | 否 | 否 |
> | operationsstatus | 否 | 否 |
> | partnernamespaces | 是 | 是 |
> | partnernamespaces / eventchannels | 否 | 否 |
> | partnerregistrations | 否 | 否 |
> | partnertopics | 是 | 是 |
> | partnertopics / eventsubscriptions | 否 | 否 |
> | systemtopics | 是 | 是 |
> | systemtopics / eventsubscriptions | 否 | 否 |
> | topics | 是 | 是 |
> | topictypes | 否 | 否 |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | availableclusterregions | 否 | 否 |
> | checknameavailability | 否 | 否 |
> | checknamespaceavailability | 否 | 否 |
> | clusters | 是 | 是 |
> | locations | 否 | 否 |
> | locations / deletevirtualnetworkorsubnets | 否 | 否 |
> | namespaces | 是 | 是 |
> | namespaces / authorizationrules | 否 | 否 |
> | namespaces / disasterrecoveryconfigs | 否 | 否 |
> | namespaces / disasterrecoveryconfigs / checknameavailability | 否 | 否 |
> | namespaces / eventhubs | 否 | 否 |
> | namespaces / eventhubs / authorizationrules | 否 | 否 |
> | namespaces / eventhubs / consumergroups | 否 | 否 |
> | namespaces / networkrulesets | 否 | 否 |
> | 操作 | 否 | 否 |
> | sku | 否 | 否 |

<!-- Not Available on ## Microsoft.Experimentation-->
<!-- Not Available on ## Microsoft.Falcon-->

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | featureproviders | 否 | 否 |
> | features | 否 | 否 |
> | 操作 | 否 | 否 |
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
> | 操作 | 否 | 否 |
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
> | clusters/applications | 否 | 否 |
> | clusters / operationresults | 否 | 否 |
> | locations | 否 | 否 |
> | locations / azureasyncoperations | 否 | 否 |
> | locations / billingspecs | 否 | 否 |
> | locations / capabilities | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / usages | 否 | 否 |
> | locations / validatecreaterequest | 否 | 否 |
> | 操作 | 否 | 否 |

<!-- Not Available on ## Microsoft.HealthcareApis-->
<!-- Not Available on ## Microsoft.HybridCompute-->
<!-- Not Available on ## Microsoft.HybridData-->
<!-- Not Available on ## Microsoft.Hydra-->
<!--Not Available on ## Microsoft.Hydra-->

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | jobs | 是 | 是 |
> | locations | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | 操作 | 否 | 否 |

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
> | calculatebaseline | 否 | 否 |
> | components | 是 | 是 |
> | components / events | 否 | 否 |
> | components / linkedstorageaccounts | 否 | 否 |
> | components/metadata | 否 | 否 |
> | components / metrics | 否 | 否 |
> | components / pricingplans | 否 | 否 |
> | components / query | 否 | 否 |
> | datacollectionrules | 否 | 否 |
> | diagnosticsettings | 否 | 否 |
> | diagnosticsettingscategories | 否 | 否 |
> | eventcategories | 否 | 否 |
> | eventtypes | 否 | 否 |
> | extendeddiagnosticsettings | 否 | 否 |
> | guestdiagnosticsettings | 否 | 否 |
> | listmigrationdate | 否 | 否 |
> | locations | 否 | 否 |
> | locations / operationresults | 否 | 否 |
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
> | 操作 | 否 | 否 |
> | privatelinkscopeoperationstatuses | 否 | 否 |
> | privatelinkscopes | 否 | 否 |
> | privatelinkscopes / privateendpointconnectionproxies | 否 | 否 |
> | privatelinkscopes / privateendpointconnections | 否 | 否 |
> | privatelinkscopes / scopedresources | 否 | 否 |
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
> | checknameavailability | 否 | 否 |
> | checksubdomainavailability | 否 | 否 |
> | iotapps | 是 | 是 |
> | 操作 | 否 | 否 |

<!-- Not Available on ## Microsoft.IoTSpaces-->

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!IMPORTANT]
> 用于磁盘加密的 Key Vault 不能移到同一订阅中的资源组，也不能跨订阅移动。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | checknameavailability | 否 | 否 |
> | deletedvaults | 否 | 否 |
> | hsmpools | 否 | 否 |
> | locations | 否 | 否 |
> | locations / deletedvaults | 否 | 否 |
> | locations / deletevirtualnetworkorsubnets | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | managedhsms | 否 | 否 |
> | 操作 | 否 | 否 |
> | vaults | 是 | 是 |
> | vaults / accesspolicies | 否 | 否 |
> | vaults / eventgridfilters | 否 | 否 |
> | vaults / secrets | 否 | 否 |

<!--Not Available on ## Microsoft.Kubernetes-->
<!-- Not Available on ## Microsoft.Kubernetes-->

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | clusters | 是 | 是 |
> | clusters / attacheddatabaseconfigurations | 否 | 否 |
> | clusters / databases | 否 | 否 |
> | clusters / databases / dataconnections | 否 | 否 |
> | clusters / databases / eventhubconnections | 否 | 否 |
> | clusters / databases / principalassignments | 否 | 否 |
> | clusters / principalassignments | 否 | 否 |
> | locations | 否 | 否 |
> | locations / checknameavailability | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | 操作 | 否 | 否 |

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
> | locations | 否 | 否 |
> | locations / workflows | 否 | 否 |
> | 操作 | 否 | 否 |
> | workflows | 是 | 是 |

<!-- Not Available on ## Microsoft.MachineLearning-->
<!-- Not Available on ## Microsoft.MachineLearningCompute-->
<!-- Not Available on ## Microsoft.MachineLearningExperimentation-->
<!-- Not Available on ## Microsoft.MachineLearningModelManagement-->

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | locations | 否 | 否 |
> | locations / computeoperationsstatus | 否 | 否 |
> | locations / quotas | 否 | 否 |
> | locations / updatequotas | 否 | 否 |
> | locations / usages | 否 | 否 |
> | locations / vmsizes | 否 | 否 |
> | locations / workspaceoperationsstatus | 否 | 否 |
> | 操作 | 否 | 否 |
> | workspaces | 否 | 否 |
> | workspaces / computes | 否 | 否 |
> | workspaces / eventgridfilters | 否 | 否 |

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | applyupdates | 否 | 否 |
> | configurationassignments | 否 | 否 |
> | maintenanceconfigurations | 是 | 是 |
> | updates | 否 | 否 |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | identities | 否 | 否 |
> | 操作 | 否 | 否 |
> | userassignedidentities | 否 | 否 |

<!-- Not Available on ## Microsoft.ManagedNetwork-->

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | marketplaceregistrationdefinitions | 否 | 否 |
> | 操作 | 否 | 否 |
> | operationstatuses | 否 | 否 |
> | registrationassignments | 否 | 否 |
> | registrationdefinitions | 否 | 否 |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | checknameavailability | 否 | 否 |
> | getentities | 否 | 否 |
> | managementgroups | 否 | 否 |
> | managementgroups / settings | 否 | 否 |
> | operationresults | 否 | 否 |
> | operationresults / asyncoperation | 否 | 否 |
> | 操作 | 否 | 否 |
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
> | checknameavailability | 否 | 否 |
> | locations | 否 | 否 |
> | locations / checknameavailability | 否 | 否 |
> | mediaservices | 是 | 是 |
> | mediaservices / accountfilters | 否 | 否 |
> | mediaservices / assets | 否 | 否 |
> | mediaservices / assets / assetfilters | 否 | 否 |
> | mediaservices / contentkeypolicies | 否 | 否 |
> | mediaservices / eventgridfilters | 否 | 否 |
> | mediaservices / liveeventoperations | 否 | 否 |
> | mediaservices / liveevents | 是 | 是 |
> | mediaservices / liveevents / liveoutputs | 否 | 否 |
> | mediaservices / liveoutputoperations | 否 | 否 |
> | mediaservices / streamingendpointoperations | 否 | 否 |
> | mediaservices / streamingendpoints | 是 | 是 |
> | mediaservices / streaminglocators | 否 | 否 |
> | mediaservices / streamingpolicies | 否 | 否 |
> | mediaservices / transforms | 否 | 否 |
> | mediaservices / transforms / jobs | 否 | 否 |
> | 操作 | 否 | 否 |

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
> | applicationgatewayavailablerequestheaders | 否 | 否 |
> | applicationgatewayavailableresponseheaders | 否 | 否 |
> | applicationgatewayavailableservervariables | 否 | 否 |
> | applicationgatewayavailablessloptions | 否 | 否 |
> | applicationgatewayavailablewafrulesets | 否 | 否 |
> | applicationgateways | 否 | 否 |
> | applicationgatewaywebapplicationfirewallpolicies | 否 | 否 |
> | applicationsecuritygroups | 是 | 是 |
> | azurefirewallfqdntags | 否 | 否 |
> | azurefirewalls | 否 | 否 |
> | bastionhosts | 否 | 否 |
> | bgpservicecommunities | 否 | 否 |
> | checkfrontdoornameavailability | 否 | 否 |
> | checktrafficmanagernameavailability | 否 | 否 |
> | connections | 是 | 是 |
> | dnsoperationresults | 否 | 否 |
> | dnsoperationstatuses | 否 | 否 |
> | dnszones | 是 | 是 |
> | dnszones / a | 否 | 否 |
> | dnszones / aaaa | 否 | 否 |
> | dnszones / all | 否 | 否 |
> | dnszones / caa | 否 | 否 |
> | dnszones / cname | 否 | 否 |
> | dnszones / mx | 否 | 否 |
> | dnszones / ns | 否 | 否 |
> | dnszones / ptr | 否 | 否 |
> | dnszones / recordsets | 否 | 否 |
> | dnszones / soa | 否 | 否 |
> | dnszones / srv | 否 | 否 |
> | dnszones / txt | 否 | 否 |
> | expressroutecircuits | 否 | 否 |
> | expressroutegateways | 否 | 否 |
> | expressrouteserviceproviders | 否 | 否 |
> | firewallpolicies | 是 | 是 |
> | getdnsresourcereference | 否 | 否 |
> | internalnotify | 否 | 否 |
> | ipallocations | 是 | 是 |
> | ipgroups | 是 | 是 |
> | loadbalancers | 是 - 基本 SKU<br />否 - 标准 SKU | 是 - 基本 SKU<br />否 - 标准 SKU |
> | localnetworkgateways | 是 | 是 |
> | locations | 否 | 否 |
> | locations / autoapprovedprivatelinkservices | 否 | 否 |
> | locations / availabledelegations | 否 | 否 |
> | locations / availableprivateendpointtypes | 否 | 否 |
> | locations / availableservicealiases | 否 | 否 |
> | locations / baremetaltenants | 否 | 否 |
> | locations / batchnotifyprivateendpointsforresourcemove | 否 | 否 |
> | locations / batchvalidateprivateendpointsforresourcemove | 否 | 否 |
> | locations / checkacceleratednetworkingsupport | 否 | 否 |
> | locations / checkdnsnameavailability | 否 | 否 |
> | locations / checkprivatelinkservicevisibility | 否 | 否 |
> | locations / commitinternalazurenetworkmanagerconfiguration | 否 | 否 |
> | locations / effectiveresourceownership | 否 | 否 |
> | locations / nfvoperationresults | 否 | 否 |
> | locations / nfvoperations | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / operations | 否 | 否 |
> | locations / servicetags | 否 | 否 |
> | locations / setresourceownership | 否 | 否 |
> | locations / supportedvirtualmachinesizes | 否 | 否 |
> | locations / usages | 否 | 否 |
> | locations / validateresourceownership | 否 | 否 |
> | locations / virtualnetworkavailableendpointservices | 否 | 否 |
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
> | 操作 | 否 | 否 |
> | p2svpngateways | 否 | 否 |
> | privatednsoperationresults | 否 | 否 |
> | privatednsoperationstatuses | 否 | 否 |
> | privatednszones | 是 | 是 |
> | privatednszones / a | 否 | 否 |
> | privatednszones / aaaa | 否 | 否 |
> | privatednszones / all | 否 | 否 |
> | privatednszones / cname | 否 | 否 |
> | privatednszones / mx | 否 | 否 |
> | privatednszones / ptr | 否 | 否 |
> | privatednszones / soa | 否 | 否 |
> | privatednszones / srv | 否 | 否 |
> | privatednszones / txt | 否 | 否 |
> | privatednszones/virtualnetworklinks | 是 | 是 |
> | privatednszonesinternal | 否 | 否 |
> | privateendpointredirectmaps | 否 | 否 |
> | privateendpoints | 是 | 是 |
> | privatelinkservices | 否 | 否 |
> | publicipaddresses | 是 - 基本 SKU<br />否 - 标准 SKU | 是 - 基本 SKU<br />否 - 标准 SKU |
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
> | checknameavailability | 否 | 否 |
> | checknamespaceavailability | 否 | 否 |
> | namespaces | 是 | 是 |
> | namespaces/notificationhubs | 是 | 是 |
> | operationresults | 否 | 否 |
> | 操作 | 否 | 否 |

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
> | locations | 否 | 否 |
> | locations / operationstatuses | 否 | 否 |
> | 操作 | 否 | 否 |
> | storageinsightconfigs | 否 | 否 |
> | workspaces | 是 | 是 |
> | workspaces / datasources | 否 | 否 |
> | workspaces / linkedservices | 否 | 否 |
> | workspaces / linkedstorageaccounts | 否 | 否 |
> | workspaces / metadata | 否 | 否 |
> | workspaces / query | 否 | 否 |
> | workspaces / scopedprivatelinkproxies | 否 | 否 |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | managementassociations | 否 | 否 |
> | managementconfigurations | 是 | 是 |
> | 操作 | 否 | 否 |
> | solutions | 是 | 是 |
> | 视图 | 是 | 是 |

<!--Not Available on ## Microsoft.Peering-->

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | asyncoperationresults | 否 | 否 |
> | 操作 | 否 | 否 |
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
> | locations | 否 | 否 |
> | locations / consoles | 否 | 否 |
> | locations / usersettings | 否 | 否 |
> | 操作 | 否 | 否 |
> | usersettings | 否 | 否 |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | locations | 否 | 否 |
> | locations / checknameavailability | 否 | 否 |
> | workspacecollections | 是 | 是 |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | capacities | 是 | 是 |
> | locations | 否 | 否 |
> | locations / checknameavailability | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / operationstatuses | 否 | 否 |
> | 操作 | 否 | 否 |

<!--Not Available on ## Microsoft.PowerPlatform-->
<!--Not Available on ## Microsoft.ProjectBabylon-->
<!--Not Available on ## Microsoft.ProviderHub-->
<!--Not Available on ## Microsoft.Quantum-->

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

<!--Not Available on [Recovery Services move guidance](../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure-resource-manager/toc.json)-->

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | backupprotecteditems | 否 | 否 |
> | locations | 否 | 否 |
> | locations / allocatedstamp | 否 | 否 |
> | locations / allocatestamp | 否 | 否 |
> | locations / backupaadproperties | 否 | 否 |
> | locations / backupcrossregionrestore | 否 | 否 |
> | locations / backupcrrjob | 否 | 否 |
> | locations / backupcrrjobs | 否 | 否 |
> | locations / backupcrroperationresults | 否 | 否 |
> | locations / backupcrroperationsstatus | 否 | 否 |
> | locations / backupprevalidateprotection | 否 | 否 |
> | locations / backupstatus | 否 | 否 |
> | locations / backupvalidatefeatures | 否 | 否 |
> | locations / checknameavailability | 否 | 否 |
> | 操作 | 否 | 否 |
> | replicationeligibilityresults | 否 | 否 |
> | vaults | 是 | 是 |

<!--Not Available on ## Microsoft.RedHatOpenShift-->

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | checknameavailability | 否 | 否 |
> | namespaces | 是 | 是 |
> | namespaces / authorizationrules | 否 | 否 |
> | namespaces / hybridconnections | 否 | 否 |
> | namespaces / hybridconnections / authorizationrules | 否 | 否 |
> | namespaces / privateendpointconnections | 否 | 否 |
> | namespaces / wcfrelays | 否 | 否 |
> | namespaces / wcfrelays / authorizationrules | 否 | 否 |
> | 操作 | 否 | 否 |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | 操作 | 否 | 否 |
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
> | availabilitystatuses | 否 | 否 |
> | childavailabilitystatuses | 否 | 否 |
> | childresources | 否 | 否 |
> | emergingissues | 否 | 否 |
> | events | 否 | 否 |
> | metadata | 否 | 否 |
> | 通知 | 否 | 否 |
> | 操作 | 否 | 否 |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | calculatetemplatehash | 否 | 否 |
> | checkpolicycompliance | 否 | 否 |
> | checkresourcename | 否 | 否 |
> | deployments | 否 | 否 |
> | deployments / operations | 否 | 否 |
> | deploymentscripts | 否 | 否 |
> | deploymentscripts / logs | 否 | 否 |
> | 链接 | 否 | 否 |
> | locations | 否 | 否 |
> | locations / deploymentscriptoperationresults | 否 | 否 |
> | notifyresourcejobs | 否 | 否 |
> | operationresults | 否 | 否 |
> | 操作 | 否 | 否 |
> | providers | 否 | 否 |
> | resourcegroups | 否 | 否 |
> | resources | 否 | 否 |
> | subscriptions | 否 | 否 |
> | subscriptions / locations | 否 | 否 |
> | subscriptions / operationresults | 否 | 否 |
> | subscriptions / providers | 否 | 否 |
> | subscriptions / resourcegroups | 否 | 否 |
> | subscriptions / resourcegroups / resources | 否 | 否 |
> | subscriptions / resources | 否 | 否 |
> | subscriptions / tagnames | 否 | 否 |
> | subscriptions / tagnames / tagvalues | 否 | 否 |
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
> | checknameavailability | 否 | 否 |
> | checkservicenameavailability | 否 | 否 |
> | 操作 | 否 | 否 |
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
> | locations | 否 | 否 |
> | locations / alerts | 否 | 否 |
> | locations / allowedconnections | 否 | 否 |
> | locations / applicationwhitelistings | 否 | 否 |
> | locations / discoveredsecuritysolutions | 否 | 否 |
> | locations / externalsecuritysolutions | 否 | 否 |
> | locations / jitnetworkaccesspolicies | 否 | 否 |
> | locations / securitysolutions | 否 | 否 |
> | locations / securitysolutionsreferencedata | 否 | 否 |
> | locations / tasks | 否 | 否 |
> | locations / topologies | 否 | 否 |
> | 操作 | 否 | 否 |
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
> | checknameavailability | 否 | 否 |
> | checknamespaceavailability | 否 | 否 |
> | locations | 否 | 否 |
> | locations / deletevirtualnetworkorsubnets | 否 | 否 |
> | namespaces | 是 | 是 |
> | namespaces / authorizationrules | 否 | 否 |
> | namespaces / disasterrecoveryconfigs | 否 | 否 |
> | namespaces / disasterrecoveryconfigs / checknameavailability | 否 | 否 |
> | namespaces / eventgridfilters | 否 | 否 |
> | namespaces / networkrulesets | 否 | 否 |
> | namespaces / queues | 否 | 否 |
> | namespaces / queues / authorizationrules | 否 | 否 |
> | namespaces / topics | 否 | 否 |
> | namespaces / topics / authorizationrules | 否 | 否 |
> | namespaces / topics / subscriptions | 否 | 否 |
> | namespaces / topics / subscriptions / rules | 否 | 否 |
> | 操作 | 否 | 否 |
> | premiummessagingregions | 否 | 否 |
> | sku | 否 | 否 |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | applications | 否 | 否 |
> | clusters | 是 | 是 |
> | clusters/applications | 否 | 否 |
> | containergroups | 否 | 否 |
> | containergroupsets | 否 | 否 |
> | edgeclusters | 否 | 否 |
> | locations | 否 | 否 |
> | locations / clusterversions | 否 | 否 |
> | locations / environments | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / operations | 否 | 否 |
> | managedclusters | 否 | 否 |
> | networks | 否 | 否 |
> | 操作 | 否 | 否 |
> | secretstores | 否 | 否 |
> | volumes | 否 | 否 |

<!-- Not Available on ## Microsoft.ServiceFabricMesh-->
<!-- Not Available on ## Microsoft.Services-->

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | locations | 否 | 否 |
> | locations / checknameavailability | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / operationstatuses | 否 | 否 |
> | locations / usages | 否 | 否 |
> | 操作 | 否 | 否 |
> | signalr | 是 | 是 |
> | signalr / eventgridfilters | 否 | 否 |

<!-- Not Available on ## Microsoft.SoftwarePlan-->

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | applicationdefinitions | 否 | 否 |
> | applications | 否 | 否 |
> | jitrequests | 否 | 否 |
> | locations | 否 | 否 |
> | locations / operationstatuses | 否 | 否 |
> | 操作 | 否 | 否 |

## <a name="microsoftsql"></a>Microsoft.Sql

> [!IMPORTANT]
> 数据库和服务器必须位于同一个资源组中。 移动 SQL 服务器时，也会移动其所有数据库。 此行为适用于 Azure SQL 数据库和 Azure Synapse Analytics 数据库。

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | checknameavailability | 否 | 否 |
> | instancepools | 否 | 否 |
> | locations | 是 | 是 |
> | locations / administratorazureasyncoperation | 否 | 否 |
> | locations / administratoroperationresults | 否 | 否 |
> | locations / auditingsettingsazureasyncoperation | 否 | 否 |
> | locations / auditingsettingsoperationresults | 否 | 否 |
> | locations / capabilities | 否 | 否 |
> | locations / databaseazureasyncoperation | 否 | 否 |
> | locations / databaseoperationresults | 否 | 否 |
> | locations / databaserestoreazureasyncoperation | 否 | 否 |
> | locations / deletevirtualnetworkorsubnets | 否 | 否 |
> | locations / deletevirtualnetworkorsubnetsazureasyncoperation | 否 | 否 |
> | locations / deletevirtualnetworkorsubnetsoperationresults | 否 | 否 |
> | locations / dnsaliasasyncoperation | 否 | 否 |
> | locations / dnsaliasoperationresults | 否 | 否 |
> | locations / elasticpoolazureasyncoperation | 否 | 否 |
> | locations / elasticpooloperationresults | 否 | 否 |
> | locations / encryptionprotectorazureasyncoperation | 否 | 否 |
> | locations / encryptionprotectoroperationresults | 否 | 否 |
> | locations / extendedauditingsettingsazureasyncoperation | 否 | 否 |
> | locations / extendedauditingsettingsoperationresults | 否 | 否 |
> | locations / failovergroupazureasyncoperation | 否 | 否 |
> | locations / failovergroupoperationresults | 否 | 否 |
> | locations / firewallrulesazureasyncoperation | 否 | 否 |
> | locations / firewallrulesoperationresults | 否 | 否 |
> | locations / instancefailovergroupazureasyncoperation | 否 | 否 |
> | locations / instancefailovergroupoperationresults | 否 | 否 |
> | locations / instancefailovergroups | 否 | 否 |
> | locations / instancepoolazureasyncoperation | 否 | 否 |
> | locations / instancepooloperationresults | 否 | 否 |
> | locations / jobagentazureasyncoperation | 否 | 否 |
> | locations / jobagentoperationresults | 否 | 否 |
> | locations / longtermretentionbackupazureasyncoperation | 否 | 否 |
> | locations / longtermretentionbackupoperationresults | 否 | 否 |
> | locations / longtermretentionbackups | 是 | 是 |
> | locations / longtermretentionmanagedinstancebackupazureasyncoperation | 否 | 否 |
> | locations / longtermretentionmanagedinstancebackupoperationresults | 否 | 否 |
> | locations / longtermretentionmanagedinstancebackups | 否 | 否 |
> | locations / longtermretentionmanagedinstances | 否 | 否 |
> | locations / longtermretentionpolicyazureasyncoperation | 否 | 否 |
> | locations / longtermretentionpolicyoperationresults | 否 | 否 |
> | locations / longtermretentionservers | 否 | 否 |
> | locations / manageddatabaseazureasyncoperation | 否 | 否 |
> | locations / manageddatabasecompleterestoreazureasyncoperation | 否 | 否 |
> | locations / manageddatabasecompleterestoreoperationresults | 否 | 否 |
> | locations / manageddatabaseoperationresults | 否 | 否 |
> | locations / manageddatabaserestoreazureasyncoperation | 否 | 否 |
> | locations / manageddatabaserestoreoperationresults | 否 | 否 |
> | locations / managedinstanceazureasyncoperation | 否 | 否 |
> | locations / managedinstanceencryptionprotectorazureasyncoperation | 否 | 否 |
> | locations / managedinstanceencryptionprotectoroperationresults | 否 | 否 |
> | locations / managedinstancekeyazureasyncoperation | 否 | 否 |
> | locations / managedinstancekeyoperationresults | 否 | 否 |
> | locations / managedinstancelongtermretentionpolicyazureasyncoperation | 否 | 否 |
> | locations / managedinstancelongtermretentionpolicyoperationresults | 否 | 否 |
> | locations / managedinstanceoperationresults | 否 | 否 |
> | locations / managedinstancetdecertazureasyncoperation | 否 | 否 |
> | locations / managedinstancetdecertoperationresults | 否 | 否 |
> | locations / managedserversecurityalertpoliciesazureasyncoperation | 否 | 否 |
> | locations / managedserversecurityalertpoliciesoperationresults | 否 | 否 |
> | locations / managedshorttermretentionpolicyazureasyncoperation | 否 | 否 |
> | locations / managedshorttermretentionpolicyoperationresults | 否 | 否 |
> | locations / notifyazureasyncoperation | 否 | 否 |
> | locations / privateendpointconnectionazureasyncoperation | 否 | 否 |
> | locations / privateendpointconnectionoperationresults | 否 | 否 |
> | locations / privateendpointconnectionproxyazureasyncoperation | 否 | 否 |
> | locations / privateendpointconnectionproxyoperationresults | 否 | 否 |
> | locations / securityalertpoliciesazureasyncoperation | 否 | 否 |
> | locations / securityalertpoliciesoperationresults | 否 | 否 |
> | locations / serveradministratorazureasyncoperation | 否 | 否 |
> | locations / serveradministratoroperationresults | 否 | 否 |
> | locations / serverazureasyncoperation | 否 | 否 |
> | locations / serverkeyazureasyncoperation | 否 | 否 |
> | locations / serverkeyoperationresults | 否 | 否 |
> | locations / serveroperationresults | 否 | 否 |
> | locations / shorttermretentionpolicyazureasyncoperation | 否 | 否 |
> | locations / shorttermretentionpolicyoperationresults | 否 | 否 |
> | locations / syncagentoperationresults | 否 | 否 |
> | locations / syncdatabaseids | 否 | 否 |
> | locations / syncgroupoperationresults | 否 | 否 |
> | locations / syncmemberoperationresults | 否 | 否 |
> | locations / tdecertazureasyncoperation | 否 | 否 |
> | locations / tdecertoperationresults | 否 | 否 |
> | locations / usages | 否 | 否 |
> | locations / virtualclusterazureasyncoperation | 否 | 否 |
> | locations / virtualclusteroperationresults | 否 | 否 |
> | locations / virtualnetworkrulesazureasyncoperation | 否 | 否 |
> | locations / virtualnetworkrulesoperationresults | 否 | 否 |
> | locations / vulnerabilityassessmentscanazureasyncoperation | 否 | 否 |
> | locations / vulnerabilityassessmentscanoperationresults | 否 | 否 |
> | managedinstances | 否 | 否 |
> | managedinstances / administrators | 否 | 否 |
> | managedinstances/databases | 否 | 否 |
> | managedinstances / databases / backuplongtermretentionpolicies | 否 | 否 |
> | managedinstances / databases / vulnerabilityassessments | 否 | 否 |
> | managedinstances / metricdefinitions | 否 | 否 |
> | managedinstances / metrics | 否 | 否 |
> | managedinstances / recoverabledatabases | 否 | 否 |
> | managedinstances / tdecertificates | 否 | 否 |
> | managedinstances / vulnerabilityassessments | 否 | 否 |
> | 操作 | 否 | 否 |
> | servers | 是 | 是 |
> | servers / administratoroperationresults | 否 | 否 |
> | servers / administrators | 否 | 否 |
> | servers / advisors | 否 | 否 |
> | servers / aggregateddatabasemetrics | 否 | 否 |
> | servers / auditingpolicies | 否 | 否 |
> | servers / auditingsettings | 否 | 否 |
> | servers / automatictuning | 否 | 否 |
> | servers / communicationlinks | 否 | 否 |
> | servers / connectionpolicies | 否 | 否 |
> | servers/databases | 是 | 是 |
> | servers / databases / advisors | 否 | 否 |
> | servers / databases / auditingpolicies | 否 | 否 |
> | servers / databases / auditingsettings | 否 | 否 |
> | servers / databases / auditrecords | 否 | 否 |
> | servers / databases / automatictuning | 否 | 否 |
> | servers / databases / backuplongtermretentionpolicies | 是 | 是 |
> | servers / databases / backupshorttermretentionpolicies | 否 | 否 |
> | servers / databases / connectionpolicies | 否 | 否 |
> | servers / databases / datamaskingpolicies | 否 | 否 |
> | servers / databases / datamaskingpolicies / rules | 否 | 否 |
> | servers / databases / extensions | 否 | 否 |
> | servers / databases / geobackuppolicies | 否 | 否 |
> | servers / databases / metricdefinitions | 否 | 否 |
> | servers / databases / metrics | 否 | 否 |
> | servers / databases / recommendedsensitivitylabels | 否 | 否 |
> | servers / databases / securityalertpolicies | 否 | 否 |
> | servers / databases / syncgroups | 否 | 否 |
> | servers / databases / syncgroups / syncmembers | 否 | 否 |
> | servers / databases / topqueries | 否 | 否 |
> | servers / databases / topqueries / querytext | 否 | 否 |
> | servers / databases / transparentdataencryption | 否 | 否 |
> | servers / databases / vulnerabilityassessment | 否 | 否 |
> | servers / databases / vulnerabilityassessments | 否 | 否 |
> | servers / databases / vulnerabilityassessmentscans | 否 | 否 |
> | servers / databases / vulnerabilityassessmentsettings | 否 | 否 |
> | servers / databases / workloadgroups | 否 | 否 |
> | servers / databasesecuritypolicies | 否 | 否 |
> | servers / disasterrecoveryconfiguration | 否 | 否 |
> | servers / dnsaliases | 否 | 否 |
> | servers / elasticpoolestimates | 否 | 否 |
> | servers / elasticpools | 是 | 是 |
> | servers / elasticpools / advisors | 否 | 否 |
> | servers / elasticpools / metricdefinitions | 否 | 否 |
> | servers / elasticpools / metrics | 否 | 否 |
> | servers / encryptionprotector | 否 | 否 |
> | servers / extendedauditingsettings | 否 | 否 |
> | servers / failovergroups | 否 | 否 |
> | servers / import | 否 | 否 |
> | servers / importexportoperationresults | 否 | 否 |
> | servers / jobaccounts | 是 | 是 |
> | servers/jobagents | 是 | 是 |
> | servers / jobagents / jobs | 否 | 否 |
> | servers / jobagents / jobs / executions | 否 | 否 |
> | servers / jobagents / jobs / steps | 否 | 否 |
> | servers / keys | 否 | 否 |
> | servers / operationresults | 否 | 否 |
> | servers / recommendedelasticpools | 否 | 否 |
> | servers / recoverabledatabases | 否 | 否 |
> | servers / restorabledroppeddatabases | 否 | 否 |
> | servers / securityalertpolicies | 否 | 否 |
> | servers / serviceobjectives | 否 | 否 |
> | servers / syncagents | 否 | 否 |
> | servers / tdecertificates | 否 | 否 |
> | servers / usages | 否 | 否 |
> | servers / virtualnetworkrules | 否 | 否 |
> | servers / vulnerabilityassessments | 否 | 否 |
> | virtualclusters | 是 | 是 |

<!-- Not Available on ## Microsoft.SqlVirtualMachine-->

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | 资源类型 | 资源组 | 订阅 |
> | ------------- | ----------- | ---------- |
> | checknameavailability | 否 | 否 |
> | locations | 否 | 否 |
> | locations / asyncoperations | 否 | 否 |
> | locations / checknameavailability | 否 | 否 |
> | locations / deletevirtualnetworkorsubnets | 否 | 否 |
> | locations / usages | 否 | 否 |
> | 操作 | 否 | 否 |
> | storageaccounts | 是 | 是 |
> | storageaccounts / blobservices | 否 | 否 |
> | storageaccounts / fileservices | 否 | 否 |
> | storageaccounts / listaccountsas | 否 | 否 |
> | storageaccounts / listservicesas | 否 | 否 |
> | storageaccounts / queueservices | 否 | 否 |
> | storageaccounts / services | 否 | 否 |
> | storageaccounts / services / metricdefinitions | 否 | 否 |
> | storageaccounts / tableservices | 否 | 否 |
> | usages | 否 | 否 |

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
> | locations | 否 | 否 |
> | locations / quotas | 否 | 否 |
> | 操作 | 否 | 否 |
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
> | environments / accesspolicies | No | No |
> | environments / eventsources | Yes | Yes |
> | environments / referencedatasets | Yes | Yes |
> | operations | No | No |

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
> | checknameavailability | 否 | 否 |
> | connectiongateways | 是 | 是 |
> | connections | 是 | 是 |
> | customapis | 是 | 是 |
> | deletedsites | 否 | 否 |
> | deploymentlocations | 否 | 否 |
> | georegions | 否 | 否 |
> | hostingenvironments | 否 | 否 |
> | hostingenvironments / eventgridfilters | 否 | 否 |
> | hostingenvironments / multirolepools | 否 | 否 |
> | hostingenvironments / workerpools | 否 | 否 |
> | ishostingenvironmentnameavailable | 否 | 否 |
> | ishostnameavailable | 否 | 否 |
> | isusernameavailable | 否 | 否 |
> | kubeenvironments | 是 | 是 |
> | listsitesassignedtohostname | 否 | 否 |
> | locations | 否 | 否 |
> | locations / apioperations | 否 | 否 |
> | locations / connectiongatewayinstallations | 否 | 否 |
> | locations / deletedsites | 否 | 否 |
> | locations / deletevirtualnetworkorsubnets | 否 | 否 |
> | locations / extractapidefinitionfromwsdl | 否 | 否 |
> | locations / getnetworkpolicies | 否 | 否 |
> | locations / listwsdlinterfaces | 否 | 否 |
> | locations / managedapis | 否 | 否 |
> | locations / operationresults | 否 | 否 |
> | locations / operations | 否 | 否 |
> | locations / runtimes | 否 | 否 |
> | 操作 | 否 | 否 |
> | publishingusers | 否 | 否 |
> | 建议 | 否 | 否 |
> | resourcehealthmetadata | 否 | 否 |
> | runtimes | 否 | 否 |
> | serverfarms | 是 | 是 |
> | serverfarms / eventgridfilters | 否 | 否 |
> | sites | 是 | 是 |
> | sites / eventgridfilters | 否 | 否 |
> | sites / hostnamebindings | 否 | 否 |
> | sites / networkconfig | 否 | 否 |
> | sites/premieraddons | 是 | 是 |
> | sites/slots | 是 | 是 |
> | sites / slots / eventgridfilters | 否 | 否 |
> | sites / slots / hostnamebindings | 否 | 否 |
> | sites / slots / networkconfig | 否 | 否 |
> | sourcecontrols | 否 | 否 |
> | staticsites | 否 | 否 |
> | validate | 否 | 否 |
> | verifyhostingenvironmentvnet | 否 | 否 |

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