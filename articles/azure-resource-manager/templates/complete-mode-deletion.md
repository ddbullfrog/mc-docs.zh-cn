---
title: 完整模式删除
description: 显示资源类型如何在 Azure 资源管理器模板中进行完全模式删除。
ms.topic: conceptual
origin.date: 09/21/2020
author: rockboyfor
ms.date: 10/12/2020
ms.testscope: yes
ms.testdate: 08/24/2020
ms.author: v-yeche
ms.openlocfilehash: 242041ef9151452a8affda560483cf3b6db711d8
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937351"
---
# <a name="deletion-of-azure-resources-for-complete-mode-deployments"></a>针对完全模式部署的 Azure 资源删除

本文描述了资源类型如何在不是以完全模式部署的模板中进行删除。

标记为 **Yes** 的资源类型不在以完全模式部署的模板中时，将会删除该类型。

标记为 **No** 的资源类型不在模板中时不会自动删除；但是，如果删除了父资源，则会删除它们。 有关此行为的完整描述，请参阅 [Azure 资源管理器部署模式](deployment-modes.md)。

如果部署到[模板中的多个资源组](cross-scope-deployment.md)，则可以删除部署操作中指定的资源组中的资源。 辅助资源组中的资源不会被删除。

跳转到资源提供程序命名空间：
> [!div class="op_single_selector"]
> - [Microsoft.AAD](#microsoftaad)
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
> - [Microsoft.CognitiveServices](#microsoftcognitiveservices)
> - [Microsoft.Compute](#microsoftcompute)
> - [Microsoft.ContainerInstance](#microsoftcontainerinstance)
> - [Microsoft.ContainerRegistry](#microsoftcontainerregistry)
> - [Microsoft.ContainerService](#microsoftcontainerservice)
> - [Microsoft.DataBox](#microsoftdatabox)
> - [Microsoft.DataFactory](#microsoftdatafactory)
> - [Microsoft.DBforMariaDB](#microsoftdbformariadb)
> - [Microsoft.DBforMySQL](#microsoftdbformysql)
> - [Microsoft.DBforPostgreSQL](#microsoftdbforpostgresql)
> - [Microsoft.Devices](#microsoftdevices)
> - [Microsoft.DocumentDB](#microsoftdocumentdb)
> - [Microsoft.EventGrid](#microsofteventgrid)
> - [Microsoft.EventHub](#microsofteventhub)
> - [Microsoft.Features](#microsoftfeatures)
> - [Microsoft.HDInsight](#microsofthdinsight)
> - [Microsoft.ImportExport](#microsoftimportexport)
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
> - [Microsoft.SQL](#microsoftsql)
> - [Microsoft.Storage](#microsoftstorage)
> - [Microsoft.StreamAnalytics](#microsoftstreamanalytics)
> - [Microsoft.TimeSeriesInsights](#microsofttimeseriesinsights)
> - [Microsoft.Web](#microsoftweb)

## <a name="microsoftaad"></a>Microsoft.AAD

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | DomainServices | 是 |
> | DomainServices / oucontainer | 否 |

<!-- Not Available on ## Microsoft.Addons-->
<!--Not Available on ## Microsoft.ADHybridHealthService-->

## <a name="microsoftadvisor"></a>Microsoft.Advisor

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | advisorScore | 否 |
> | 配置 | 否 |
> | generateRecommendations | 否 |
> | metadata | 否 |
> | 建议 | 否 |
> | 禁止显示 | 否 |

<!--Not Available on ## Microsoft.AgFoodPlatform-->

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | actionRules | 是 |
> | alerts | 否 |
> | alertsList | 否 |
> | alertsMetaData | 否 |
> | alertsSummary | 否 |
> | alertsSummaryList | 否 |
> | smartDetectorAlertRules | 是 |
> | smartGroups | 否 |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | servers | 是 |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | reportFeedback | 否 |
> | 服务 | 是 |
> | validateServiceName | 否 |

<!--Not Available on ## Microsoft.AppConfiguration-->
<!--Not Available on ## Microsoft.AppPlatform-->
<!--Not Available on ## Microsoft.Attestation-->

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | accessReviewScheduleDefinitions | 否 |
> | accessReviewScheduleSettings | 否 |
> | classicAdministrators | 否 |
> | dataAliases | 否 |
> | denyAssignments | 否 |
> | elevateAccess | 否 |
> | findOrphanRoleAssignments | 否 |
> | 锁定 | 否 |
> | 权限 | 否 |
> | policyAssignments | 否 |
> | policyDefinitions | 否 |
> | policyExemptions | 否 |
> | policySetDefinitions | 否 |
> | privateLinkAssociations | 否 |
> | providerOperations | 否 |
> | resourceManagementPrivateLinks | 是 |
> | roleAssignments | 否 |
> | roleAssignmentsUsageMetrics | 否 |
> | roleDefinitions | 否 |

<!--Not Available on ## Microsoft.Automanage-->

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | automationAccounts | 是 |
> | automationAccounts / configurations | 是 |
> | automationAccounts / jobs | 否 |
> | automationAccounts / privateEndpointConnectionProxies | 否 |
> | automationAccounts / privateEndpointConnections | 否 |
> | automationAccounts / privateLinkResources | 否 |
> | automationAccounts / runbooks | 是 |
> | automationAccounts / softwareUpdateConfigurations | 否 |
> | automationAccounts / webhooks | 否 |

<!--Not Available on ## Microsoft.AVS-->
<!--Not Available on ## Microsoft.Azure.Geneva-->

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | b2cDirectories | 是 |
> | b2ctenants | 否 |
> | guestUsages | 是 |

<!--Not Available on ## Microsoft.AzureData-->

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | cloudManifestFiles | 否 |
> | edgeSubscriptions | 是 |
> | linkedSubscriptions | 是 |
> | registrations | 是 |
> | registrations / customerSubscriptions | 否 |
> | registrations / products | 否 |

<!--Not Available on ## Microsoft.AzureStackHCI-->
<!--Not Available on ## Microsoft.AzureStackResourceMonitor-->

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | batchAccounts | 是 |

<!--Not Available on ## Microsoft.Billing-->
<!--Not Available on ## Microsoft.BingMaps-->
<!--Not Available on ## Microsoft.Blockchain-->
<!--Not Available on ## Microsoft.BlockchainTokens-->

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | blueprintAssignments | 否 |
> | blueprintAssignments / assignmentOperations | 否 |
> | blueprintAssignments / operations | 否 |
> | blueprints | 否 |
> | blueprints / artifacts | 否 |
> | blueprints / versions | 否 |
> | blueprints / versions / artifacts | 否 |

<!--Not Available on ## Microsoft.BotService-->

## <a name="microsoftcache"></a>Microsoft.Cache

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | Redis | 是 |
> | Redis/EventGridFilters | 否 |
> | Redis/privateEndpointConnectionProxies | 否 |
> | Redis/privateEndpointConnectionProxies/validate | 否 |
> | Redis/privateEndpointConnections | 否 |
> | Redis/privateLinkResources | 否 |
> | redisEnterprise | 是 |
> | RedisEnterprise / privateEndpointConnectionProxies | 否 |
> | RedisEnterprise / privateEndpointConnectionProxies / validate | 否 |
> | RedisEnterprise / privateEndpointConnections | 否 |
> | RedisEnterprise / privateLinkResources | 否 |

<!--Not Available on ## Microsoft.Capacity-->

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | CdnWebApplicationFirewallManagedRuleSets | 否 |
> | CdnWebApplicationFirewallPolicies | 是 |
> | edgenodes | 否 |
> | 配置文件 | 是 |
> | profiles/endpoints | 是 |
> | profiles / endpoints / customdomains | 否 |
> | profiles / endpoints / origingroups | 否 |
> | profiles / endpoints / origins | 否 |
> | validateProbe | 否 |

<!-- Not Available on ## Microsoft.CertificateRegistration-->

<!-- Not Available on ## Microsoft.ChangeAnalysis-->

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | capabilities | 否 |
> | domainNames | 是 |
> | domainNames / capabilities | 否 |
> | domainNames / internalLoadBalancers | 否 |
> | domainNames / serviceCertificates | 否 |
> | domainNames / slots | 否 |
> | domainNames / slots / roles | 否 |
> | domainNames / slots / roles / metricDefinitions | 否 |
> | domainNames / slots / roles / metrics | 否 |
> | moveSubscriptionResources | 否 |
> | operatingSystemFamilies | 否 |
> | operatingSystems | 否 |
> | quotas | 否 |
> | resourceTypes | 否 |
> | validateSubscriptionMoveAvailability | 否 |
> | virtualMachines | 是 |
> | virtualMachines / diagnosticSettings | 否 |
> | virtualMachines / metricDefinitions | 否 |
> | virtualMachines / metrics | 否 |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | classicInfrastructureResources | 否 |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | capabilities | 否 |
> | expressRouteCrossConnections | 否 |
> | expressRouteCrossConnections / peerings | 否 |
> | gatewaySupportedDevices | 否 |
> | networkSecurityGroups | 是 |
> | quotas | 否 |
> | reservedIps | 是 |
> | virtualNetworks | 是 |
> | virtualNetworks / remoteVirtualNetworkPeeringProxies | 否 |
> | virtualNetworks / virtualNetworkPeerings | 否 |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | capabilities | 否 |
> | disks | 否 |
> | images | 否 |
> | osImages | 否 |
> | osPlatformImages | 否 |
> | publicImages | 否 |
> | quotas | 否 |
> | storageAccounts | 是 |
> | storageAccounts / blobServices | 否 |
> | storageAccounts / fileServices | 否 |
> | storageAccounts / metricDefinitions | 否 |
> | storageAccounts / metrics | 否 |
> | storageAccounts / queueServices | 否 |
> | storageAccounts / services | 否 |
> | storageAccounts / services / diagnosticSettings | 否 |
> | storageAccounts / services / metricDefinitions | 否 |
> | storageAccounts / services / metrics | 否 |
> | storageAccounts / tableServices | 否 |
> | storageAccounts / vmImages | 否 |
> | vmImages | 否 |

<!--Not Available on ## Microsoft.Codespaces-->
## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | accounts | 是 |
> | accounts / privateEndpointConnectionProxies | 否 |
> | accounts / privateEndpointConnections | 否 |
> | accounts / privateLinkResources | 否 |

<!--Not Available on ## Microsoft.Commerce-->

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | availabilitySets | 是 |
> | cloudServices | 是 |
> | cloudServices / networkInterfaces | 否 |
> | cloudServices / publicIPAddresses | 否 |
> | cloudServices / roleInstances | 否 |
> | cloudServices / roleInstances / networkInterfaces | 否 |
> | cloudServices / roles | 否 |
> | diskAccesses | 是 |
> | diskEncryptionSets | 是 |
> | disks | 是 |
> | galleries | 是 |
> | galleries / applications | 否 |
> | galleries / applications / versions | 否 |
> | galleries/images | 否 |
> | galleries/images/versions | 否 |
> | hostGroups | 是 |
> | hostGroups / hosts | 是 |
> | images | 是 |
> | proximityPlacementGroups | 是 |
> | restorePointCollections | 是 |
> | restorePointCollections / restorePoints | 否 |
> | sharedVMExtensions | 是 |
> | sharedVMExtensions / versions | 否 |
> | sharedVMImages | 是 |
> | sharedVMImages / versions | 否 |
> | snapshots | 是 |
> | sshPublicKeys | 是 |
> | virtualMachines | 是 |
> | virtualMachines / extensions | 是 |
> | virtualMachines / metricDefinitions | 否 |
> | virtualMachines/runCommands | 是 |
> | virtualMachineScaleSets | 是 |
> | virtualMachineScaleSets / extensions | 否 |
> | virtualMachineScaleSets / networkInterfaces | 否 |
> | virtualMachineScaleSets / publicIPAddresses | 否 |
> | virtualMachineScaleSets / virtualMachines | 否 |
> | virtualMachineScaleSets / virtualMachines / networkInterfaces | 否 |

<!--Not Available on ## Microsoft.ConnectedCache-->
<!--Not Available on ## Microsoft.Consumption-->

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | containerGroups | 是 |
> | serviceAssociationLinks | 否 |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | registries | 是 |
> | registries / agentPools | 是 |
> | registries / builds | 否 |
> | registries / builds / cancel | 否 |
> | registries / builds / getLogLink | 否 |
> | registries / buildTasks | 是 |
> | registries / buildTasks / steps | 否 |
> | registries / eventGridFilters | 否 |
> | registries/exportPipelines | 否 |
> | registries / generateCredentials | 否 |
> | registries / getBuildSourceUploadUrl | 否 |
> | registries / GetCredentials | 否 |
> | registries / importImage | 否 |
> | registries/importPipelines | 否 |
> | registries/pipelineRuns | 否 |
> | registries / privateEndpointConnectionProxies | 否 |
> | registries / privateEndpointConnectionProxies / validate | 否 |
> | registries / privateEndpointConnections | 否 |
> | registries / privateLinkResources | 否 |
> | registries / queueBuild | 否 |
> | registries / regenerateCredential | 否 |
> | registries / regenerateCredentials | 否 |
> | registries/replications | 是 |
> | registries / runs | 否 |
> | registries / runs / cancel | 否 |
> | registries / scheduleRun | 否 |
> | registries / scopeMaps | 否 |
> | registries / taskRuns | 否 |
> | registries/tasks | 是 |
> | registries / tokens | 否 |
> | registries / updatePolicies | 否 |
> | registries/webhooks | 是 |
> | registries / webhooks / getCallbackConfig | 否 |
> | registries / webhooks / ping | 否 |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | containerServices | 是 |
> | managedClusters | 是 |
> | openShiftManagedClusters | 是 |

<!--Not Available on ## Microsoft.CostManagement-->
<!--Not Available on ## Microsoft.CustomerLockbox-->
<!--Not Available on ## Microsoft.CustomProviders-->
<!--Not Available on ## Microsoft.D365CustomerInsights-->

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | jobs | 是 |

<!--Not Available on ## Microsoft.DataBoxEdge-->
<!--Not Available on ## Microsoft.Databricks-->
<!--Not Available on ## Microsoft.DataCatalog-->

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | dataFactories | 是 |
> | dataFactories / diagnosticSettings | 否 |
> | dataFactories / metricDefinitions | 否 |
> | dataFactorySchema | 否 |
> | factories | 是 |
> | factories / integrationRuntimes | 否 |

<!--Not Available on ## Microsoft.DataLakeAnalytics-->
<!--Not Available on ## Microsoft.DataLakeStore-->

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | services | 是 |
> | services/projects | 是 |

<!--Not Available on ## Microsoft.DataProtection-->
<!--Not Available on ## Microsoft.DataShare-->

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | servers | 是 |
> | servers / advisors | 否 |
> | servers / keys | 否 |
> | servers / privateEndpointConnectionProxies | 否 |
> | servers / privateEndpointConnections | 否 |
> | servers / privateLinkResources | 否 |
> | servers / queryTexts | 否 |
> | servers / recoverableServers | 否 |
> | servers / start | 否 |
> | servers / stop | 否 |
> | servers / topQueryStatistics | 否 |
> | servers / virtualNetworkRules | 否 |
> | servers / waitStatistics | 否 |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | flexibleServers | 是 |
> | servers | 是 |
> | servers / advisors | 否 |
> | servers / keys | 否 |
> | servers / privateEndpointConnectionProxies | 否 |
> | servers / privateEndpointConnections | 否 |
> | servers / privateLinkResources | 否 |
> | servers / queryTexts | 否 |
> | servers / recoverableServers | 否 |
> | servers / start | 否 |
> | servers / stop | 否 |
> | servers / topQueryStatistics | 否 |
> | servers / upgrade | 否 |
> | servers / virtualNetworkRules | 否 |
> | servers / waitStatistics | 否 |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | flexibleServers | 是 |
> | serverGroups | 是 |
> | servers | 是 |
> | servers / advisors | 否 |
> | servers / keys | 否 |
> | servers / privateEndpointConnectionProxies | 否 |
> | servers / privateEndpointConnections | 否 |
> | servers / privateLinkResources | 否 |
> | servers / queryTexts | 否 |
> | servers / recoverableServers | 否 |
> | servers / topQueryStatistics | 否 |
> | servers / virtualNetworkRules | 否 |
> | servers / waitStatistics | 否 |
> | serversv2 | 是 |

<!--Not Available on ## Microsoft.DeploymentManager-->
<!--Not Available on ## Microsoft.DesktopVirtualization-->

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | ElasticPools | 是 |
> | ElasticPools / IotHubTenants | 是 |
> | ElasticPools / IotHubTenants / securitySettings | 否 |
> | IotHubs | 是 |
> | IotHubs / eventGridFilters | 否 |
> | IotHubs / securitySettings | 否 |
> | ProvisioningServices | 是 |
> | usages | 否 |

<!--Not Available on ## Microsoft.DeviceUpdate-->
<!--Not Available on ## Microsoft.DevOps-->
<!--Not Available on ## Microsoft.DevSpaces-->
<!--Not Available on ## Microsoft.DevTestLab-->
<!--Not Available on ## Microsoft.DigitalTwins-->

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | databaseAccountNames | 否 |
> | databaseAccounts | 是 |
> | restorableDatabaseAccounts | 否 |

<!--Not Available on ## Microsoft.DomainRegistration-->
<!--Not Available on ## Microsoft.DynamicsLcs-->
<!--Not Available on ## Microsoft.EnterpriseKnowledgeGraph--->


## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | domains | 是 |
> | domains / topics | 否 |
> | eventSubscriptions | 否 |
> | extensionTopics | 否 |
> | partnerNamespaces | 是 |
> | partnerNamespaces / eventChannels | 否 |
> | partnerRegistrations | 是 |
> | partnerTopics | 是 |
> | partnerTopics / eventSubscriptions | 否 |
> | systemTopics | 是 |
> | systemTopics / eventSubscriptions | 否 |
> | topics | 是 |
> | topicTypes | 否 |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | clusters | 是 |
> | namespaces | 是 |
> | namespaces / authorizationrules | 否 |
> | namespaces / disasterrecoveryconfigs | 否 |
> | namespaces / eventhubs | 否 |
> | namespaces / eventhubs / authorizationrules | 否 |
> | namespaces / eventhubs / consumergroups | 否 |
> | namespaces / networkrulesets | 否 |

<!-- Not Available on ## Microsoft.Experimentation-->
<!-- Not Available on ## Microsoft.Falcon-->

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | featureProviders | 否 |
> | features | 否 |
> | providers | 否 |
> | subscriptionFeatureRegistrations | 否 |

<!--Not Available on ## Microsoft.Gallery-->
<!--Not Available on ## Microsoft.Genomics-->

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | autoManagedAccounts | 是 |
> | autoManagedVmConfigurationProfiles | 是 |
> | configurationProfileAssignments | 否 |
> | guestConfigurationAssignments | 否 |
> | software | 否 |
> | softwareUpdateProfile | 否 |
> | softwareUpdates | 否 |

<!--Not Available on ## Microsoft.HanaOnAzure-->
<!--Not Available on ## Microsoft.HardwareSecurityModules-->

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | clusters | 是 |
> | clusters/applications | 否 |

<!--Not Available on ## Microsoft.HealthcareApis-->
<!--Not Available on ## Microsoft.HybridCompute-->
<!--Not Available on ## Microsoft.HybridData-->
<!--Not Available on ## Microsoft.HybridNetwork-->
<!--Not Available on ## Microsoft.Hydra-->

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | jobs | 是 |

<!-- Not Available on ## Microsoft.Intune-->

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | appTemplates | 否 |
> | IoTApps | 是 |

<!--Not Available on ## Microsoft.IoTSpaces-->

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | deletedVaults | 否 |
> | vaults | 是 |
> | vaults / accessPolicies | 否 |
> | vaults / eventGridFilters | 否 |
> | vaults / keys | 否 |
> | vaults / keys / versions | 否 |
> | vaults / secrets | 否 |

<!--Not Available on > | hsmPools | Yes |-->
<!--Not Available on > | managedHSMs | Yes |-->
<!-- Not Available on ## Microsoft.Kubernetes-->
<!-- Not Available on ## Microsoft.KubernetesConfiguration-->

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | clusters | 是 |
> | clusters / attacheddatabaseconfigurations | 否 |
> | clusters / databases | 否 |
> | clusters / databases / dataconnections | 否 |
> | clusters / databases / eventhubconnections | 否 |
> | clusters / databases / principalassignments | 否 |
> | clusters / dataconnections | 否 |
> | clusters / principalassignments | 否 |
> | clusters / sharedidentities | 否 |

<!--Not Available on ## Microsoft.LabServices-->

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | hostingEnvironments | 是 |
> | integrationAccounts | 是 |
> | integrationServiceEnvironments | 是 |
> | integrationServiceEnvironments / managedApis | 是 |
> | isolatedEnvironments | 是 |
> | workflows | 是 |

<!--Not Available on ## Microsoft.MachineLearning-->

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | workspaces | 是 |
> | workspaces / computes | 否 |
> | workspaces / eventGridFilters | 否 |
> | workspaces / linkedServices | 否 |

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | applyUpdates | 否 |
> | configurationAssignments | 否 |
> | maintenanceConfigurations | 是 |
> | publicMaintenanceConfigurations | 否 |
> | updates | 否 |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | 标识 | 否 |
> | userAssignedIdentities | 是 |

<!--Not Available on ## Microsoft.ManagedNetwork-->

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | marketplaceRegistrationDefinitions | 否 |
> | registrationAssignments | 否 |
> | registrationDefinitions | 否 |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | getEntities | 否 |
> | managementGroups | 否 |
> | managementGroups / settings | 否 |
> | resources | 否 |
> | startTenantBackfill | 否 |
> | tenantBackfillStatus | 否 |

<!--Not Available on ## Microsoft.Maps-->
<!--Not Available on ## Microsoft.Marketplace-->
<!--Not Available on ## Microsoft.MarketplaceApps-->
<!--Not Available on ## Microsoft.MarketplaceOrdering-->

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | mediaservices | 是 |
> | mediaservices / accountFilters | 否 |
> | mediaservices / assets | 否 |
> | mediaservices / assets / assetFilters | 否 |
> | mediaservices / contentKeyPolicies | 否 |
> | mediaservices / eventGridFilters | 否 |
> | mediaservices / liveEventOperations | 否 |
> | mediaservices / liveEvents | 是 |
> | mediaservices / liveEvents / liveOutputs | 否 |
> | mediaservices / liveOutputOperations | 否 |
> | mediaservices / mediaGraphs | 否 |
> | mediaservices / privateEndpointConnectionOperations | 否 |
> | mediaservices / privateEndpointConnectionProxies | 否 |
> | mediaservices / privateEndpointConnections | 否 |
> | mediaservices / streamingEndpointOperations | 否 |
> | mediaservices / streamingEndpoints | 是 |
> | mediaservices / streamingLocators | 否 |
> | mediaservices / streamingPolicies | 否 |
> | mediaservices / transforms | 否 |
> | mediaservices / transforms / jobs | 否 |

<!--Not Available on ## Microsoft.Microservices4Spring-->
<!--Not Available on ## Microsoft.Migrate-->
<!--Not Available on ## Microsoft.MixedReality-->
<!-- Not Available on ## Microsoft.NetApp-->

<!--MOONCAKE CUSTOMIZATION-->
<!--Not Available on DDOS, FRONT DOORS, PRIVATE LINK-->
## <a name="microsoftnetwork"></a>Microsoft.Network

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | applicationGateways | 是 |
> | applicationGatewayWebApplicationFirewallPolicies | 是 |
> | applicationSecurityGroups | 是 |
> | azureFirewallFqdnTags | 否 |
> | azureFirewalls | 是 |
> | bastionHosts | 是 |
> | bgpServiceCommunities | 否 |
> | connections | 是 |
> | dnsOperationStatuses | 否 |
> | dnszones | 是 |
> | dnszones / A | 否 |
> | dnszones / AAAA | 否 |
> | dnszones / all | 否 |
> | dnszones / CAA | 否 |
> | dnszones / CNAME | 否 |
> | dnszones / MX | 否 |
> | dnszones / NS | 否 |
> | dnszones / PTR | 否 |
> | dnszones / recordsets | 否 |
> | dnszones / SOA | 否 |
> | dnszones / SRV | 否 |
> | dnszones / TXT | 否 |
> | expressRouteCircuits | 是 |
> | expressRouteCrossConnections | 是 |
> | expressRouteGateways | 是 |
> | expressRoutePorts | 是 |
> | expressRouteServiceProviders | 否 |
> | firewallPolicies | 是 |
> | getDnsResourceReference | 否 |
> | internalNotify | 否 |
> | loadBalancers | 是 |
> | localNetworkGateways | 是 |
> | natGateways | 是 |
> | networkIntentPolicies | 是 |
> | networkInterfaces | 是 |
> | networkProfiles | 是 |
> | networkSecurityGroups | 是 |
> | networkWatchers | 是 |
> | networkWatchers / connectionMonitors | 是 |
> | networkWatchers / flowLogs | 是 |
> | networkWatchers / lenses | 是 |
> | networkWatchers / pingMeshes | 是 |
> | p2sVpnGateways | 是 |
> | privateDnsOperationStatuses | 否 |
> | privateDnsZones | 是 |
> | privateDnsZones / A | 否 |
> | privateDnsZones / AAAA | 否 |
> | privateDnsZones / all | 否 |
> | privateDnsZones / CNAME | 否 |
> | privateDnsZones / MX | 否 |
> | privateDnsZones / PTR | 否 |
> | privateDnsZones / SOA | 否 |
> | privateDnsZones / SRV | 否 |
> | privateDnsZones / TXT | 否 |
> | privateDnsZones / virtualNetworkLinks | 是 |
> | privateEndpoints | 是 |
> | privateLinkServices | 是 |
> | publicIPAddresses | 是 |
> | publicIPPrefixes | 是 |
> | routeFilters | 是 |
> | routeTables | 是 |
> | serviceEndpointPolicies | 是 |
> | trafficManagerGeographicHierarchies | 否 |
> | trafficmanagerprofiles | 是 |
> | trafficmanagerprofiles / heatMaps | 否 |
> | trafficManagerUserMetricsKeys | 否 |
> | virtualHubs | 是 |
> | virtualNetworkGateways | 是 |
> | virtualNetworks | 是 |
> | virtualNetworks/subnets | 否 |
> | virtualNetworkTaps | 是 |
> | virtualWans | 是 |
> | vpnGateways | 是 |
> | vpnSites | 是 |
> | webApplicationFirewallPolicies | 是 |

<!--MOONCAKE CUSTOMIZATION-->
<!--Not Available on DDOS, FRONT DOORS, PRIVATE LINK-->
<!--Not Available on ## Microsoft.Notebooks-->

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | namespaces | 是 |
> | namespaces / notificationHubs | 是 |

<!--Not Available on ## Microsoft.ObjectStore-->
<!--Not Available on ## Microsoft.OffAzure-->

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | clusters | 是 |
> | deletedWorkspaces | 否 |
> | linkTargets | 否 |
> | storageInsightConfigs | 否 |
> | workspaces | 是 |
> | workspaces / dataExports | 否 |
> | workspaces / dataSources | 否 |
> | workspaces / linkedServices | 否 |
> | workspaces / linkedStorageAccounts | 否 |
> | workspaces / metadata | 否 |
> | workspaces / query | 否 |
> | workspaces / scopedPrivateLinkProxies | 否 |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | managementassociations | 否 |
> | managementconfigurations | 是 |
> | solutions | 是 |
> | 视图 | 是 |

<!--Not Available on ## Microsoft.Peering-->

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | attestations | 否 |
> | policyEvents | 否 |
> | policyMetadata | 否 |
> | policyStates | 否 |
> | policyTrackedResources | 否 |
> | remediations | 否 |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | consoles | 否 |
> | dashboards | 是 |
> | userSettings | 否 |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | tenants | 是 |
> | tenants / workspaces | 否 |
> | workspaceCollections | 是 |

<!--Not Available on privateLinkService -->

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | capacities | 是 |

<!--Not Available on ## Microsoft.ProjectBabylon-->
<!--Not Available on ## Microsoft.ProviderHub-->
<!--Not Available on ## Microsoft.Quantum-->

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | backupProtectedItems | 否 |
> | vaults | 是 |

<!--Not Available on## Microsoft.RedHatOpenShift-->

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | namespaces | 是 |
> | namespaces / authorizationrules | 否 |
> | namespaces / hybridconnections | 否 |
> | namespaces / hybridconnections / authorizationrules | 否 |
> | namespaces / privateEndpointConnections | 否 |
> | namespaces / wcfrelays | 否 |
> | namespaces / wcfrelays / authorizationrules | 否 |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | 查询 | 是 |
> | resourceChangeDetails | 否 |
> | resourceChanges | 否 |
> | resources | 否 |
> | resourcesHistory | 否 |
> | subscriptionsStatus | 否 |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | availabilityStatuses | 否 |
> | childAvailabilityStatuses | 否 |
> | childResources | 否 |
> | emergingissues | 否 |
> | events | 否 |
> | impactedResources | 否 |
> | metadata | 否 |
> | 通知 | 否 |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | calculateTemplateHash | 否 |
> | deployments | 否 |
> | deployments / operations | 否 |
> | deploymentScripts | 是 |
> | deploymentScripts / logs | 否 |
> | 链接 | 否 |
> | notifyResourceJobs | 否 |
> | providers | 否 |
> | resourceGroups | 否 |
> | subscriptions | 否 |
> | templateSpecs | 是 |
> | templateSpecs / versions | 是 |
> | tenants | 否 |

<!--Not Available on ## Microsoft.SaaS-->
<!--Not Available on ## Microsoft.ScVmm-->

## <a name="microsoftsearch"></a>Microsoft.Search

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | resourceHealthMetadata | 否 |
> | searchServices | 是 |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | adaptiveNetworkHardenings | 否 |
> | advancedThreatProtectionSettings | 否 |
> | alerts | 否 |
> | alertsSuppressionRules | 否 |
> | allowedConnections | 否 |
> | applicationWhitelistings | 否 |
> | assessmentMetadata | 否 |
> | assessments | 否 |
> | autoDismissAlertsRules | 否 |
> | automations | 是 |
> | AutoProvisioningSettings | 否 |
> | Compliances | 否 |
> | 连接器 | 否 |
> | dataCollectionAgents | 否 |
> | deviceSecurityGroups | 否 |
> | discoveredSecuritySolutions | 否 |
> | externalSecuritySolutions | 否 |
> | InformationProtectionPolicies | 否 |
> | iotDefenderSettings | 否 |
> | iotSecuritySolutions | 是 |
> | iotSecuritySolutions / analyticsModels | 否 |
> | iotSecuritySolutions / analyticsModels / aggregatedAlerts | 否 |
> | iotSecuritySolutions / analyticsModels / aggregatedRecommendations | 否 |
> | iotSecuritySolutions / iotAlerts | 否 |
> | iotSecuritySolutions / iotAlertTypes | 否 |
> | iotSecuritySolutions / iotRecommendations | 否 |
> | iotSecuritySolutions / iotRecommendationTypes | 否 |
> | iotSensors | 否 |
> | jitNetworkAccessPolicies | 否 |
> | jitPolicies | 否 |
> | 策略 | 否 |
> | pricings | 否 |
> | regulatoryComplianceStandards | 否 |
> | regulatoryComplianceStandards / regulatoryComplianceControls | 否 |
> | regulatoryComplianceStandards / regulatoryComplianceControls / regulatoryComplianceAssessments | 否 |
> | secureScoreControlDefinitions | 否 |
> | secureScoreControls | 否 |
> | secureScores | 否 |
> | secureScores / secureScoreControls | 否 |
> | securityContacts | 否 |
> | securitySolutions | 否 |
> | securitySolutionsReferenceData | 否 |
> | securityStatuses | 否 |
> | securityStatusesSummaries | 否 |
> | serverVulnerabilityAssessments | 否 |
> | 设置 | 否 |
> | sqlVulnerabilityAssessments | 否 |
> | subAssessments | 否 |
> | 任务 | 否 |
> | topologies | 否 |
> | workspaceSettings | 否 |

<!-- Not Available on ## Microsoft.SecurityGraph-->
<!-- Not Available on ## Microsoft.SecurityInsights-->
<!--Not Available on ## Microsoft.SerialConsole-->

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | namespaces | 是 |
> | namespaces / authorizationrules | 否 |
> | namespaces / disasterrecoveryconfigs | 否 |
> | namespaces / eventgridfilters | 否 |
> | namespaces / networkrulesets | 否 |
> | namespaces / privateEndpointConnections | 否 |
> | namespaces / queues | 否 |
> | namespaces / queues / authorizationrules | 否 |
> | namespaces / topics | 否 |
> | namespaces / topics / authorizationrules | 否 |
> | namespaces / topics / subscriptions | 否 |
> | namespaces / topics / subscriptions / rules | 否 |
> | premiumMessagingRegions | 否 |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | applications | 是 |
> | clusters | 是 |
> | clusters/applications | 否 |
> | containerGroups | 是 |
> | containerGroupSets | 是 |
> | edgeclusters | 是 |
> | edgeclusters / applications | 否 |
> | managedclusters | 是 |
> | managedclusters / nodetypes | 否 |
> | networks | 是 |
> | secretstores | 是 |
> | secretstores / certificates | 否 |
> | secretstores / secrets | 否 |
> | volumes | 是 |

<!-- Not Available on ## Microsoft.ServiceFabricMesh-->
<!-- Not Available on ## Microsoft.Services-->

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | SignalR | 是 |
> | SignalR / eventGridFilters | 否 |

<!-- Not Available on ## Microsoft.SoftwarePlan-->

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | applicationDefinitions | 是 |
> | applications | 是 |
> | jitRequests | 是 |

## <a name="microsoftsql"></a>Microsoft.SQL

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | managedInstances | 是 |
> | managedInstances / databases | 是 |
> | managedInstances / databases / backupShortTermRetentionPolicies | 否 |
> | managedInstances / databases / schemas / tables / columns / sensitivityLabels | 否 |
> | managedInstances / databases / vulnerabilityAssessments | 否 |
> | managedInstances / databases / vulnerabilityAssessments / rules / baselines | 否 |
> | managedInstances / encryptionProtector | 否 |
> | managedInstances / keys | 否 |
> | managedInstances / restorableDroppedDatabases / backupShortTermRetentionPolicies | 否 |
> | managedInstances / vulnerabilityAssessments | 否 |
> | servers | 是 |
> | servers / administrators | 否 |
> | servers / communicationLinks | 否 |
> | servers/databases | 是 |
> | servers / encryptionProtector | 否 |
> | servers / firewallRules | 否 |
> | servers / keys | 否 |
> | servers / restorableDroppedDatabases | 否 |
> | servers / serviceobjectives | 否 |
> | servers / tdeCertificates | 否 |
> | virtualClusters | 否 |

<!-- Not Available on ## Microsoft.SqlVirtualMachine-->

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | deletedAccounts | 否 |
> | storageAccounts | 是 |
> | storageAccounts / blobServices | 否 |
> | storageAccounts / fileServices | 否 |
> | storageAccounts / queueServices | 否 |
> | storageAccounts / services | 否 |
> | storageAccounts / services / metricDefinitions | 否 |
> | storageAccounts / tableServices | 否 |
> | usages | 否 |

<!-- Not Available on ## Microsoft.StorageCache-->
<!-- Not Available on ## Microsoft.StorageReplication-->
<!-- Not Available on ## Microsoft.StorageSync-->
<!-- Not Available on ## Microsoft.StorageSyncDev-->
<!-- Not Available on ## Microsoft.StorageSyncInt-->
<!-- Not Available on ## Microsoft.StorSimple-->

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | clusters | 是 |
> | streamingjobs | 是 |

<!-- Not Available on ## Microsoft.Subscription-->
<!-- Not Available on ## Microsoft.Synapse-->

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | environments | 是 |
> | environments / accessPolicies | 否 |
> | environments / eventsources | 是 |
> | environments / referenceDataSets | 是 |

<!-- Not Available on ## Microsoft.Token-->
<!-- Not Available on ## Microsoft.VirtualMachineImages-->
<!-- Not Available on ## Microsoft.VMware-->
<!-- Not Available on ## Microsoft.VMwareCloudSimple-->
<!-- Not Available on ## Microsoft.VMwareOnAzure-->
<!-- Not Available on ## Microsoft.VnfManager-->
<!-- Not Available on ## Microsoft.VSOnline-->

## <a name="microsoftweb"></a>Microsoft.Web

> [!div class="mx-tableFixed"]
> | 资源类型 | 完整模式删除 |
> | ------------- | ----------- |
> | apiManagementAccounts | 否 |
> | apiManagementAccounts / apiAcls | 否 |
> | apiManagementAccounts / apis | 否 |
> | apiManagementAccounts / apis / apiAcls | 否 |
> | apiManagementAccounts / apis / connectionAcls | 否 |
> | apiManagementAccounts / apis / connections | 否 |
> | apiManagementAccounts / apis / connections / connectionAcls | 否 |
> | apiManagementAccounts / apis / localizedDefinitions | 否 |
> | apiManagementAccounts / connectionAcls | 否 |
> | apiManagementAccounts / connections | 否 |
> | billingMeters | 否 |
> | certificates | 是 |
> | connectionGateways | 是 |
> | connections | 是 |
> | customApis | 是 |
> | deletedSites | 否 |
> | hostingEnvironments | 是 |
> | hostingEnvironments / eventGridFilters | 否 |
> | hostingEnvironments / multiRolePools | 否 |
> | hostingEnvironments / workerPools | 否 |
> | kubeEnvironments | 是 |
> | publishingUsers | 否 |
> | 建议 | 否 |
> | resourceHealthMetadata | 否 |
> | runtimes | 否 |
> | serverFarms | 是 |
> | serverFarms / eventGridFilters | 否 |
> | sites | 是 |
> | sites/config  | 否 |
> | sites / eventGridFilters | 否 |
> | sites / hostNameBindings | 否 |
> | sites / networkConfig | 否 |
> | sites/premieraddons | 是 |
> | sites/slots | 是 |
> | sites / slots / eventGridFilters | 否 |
> | sites / slots / hostNameBindings | 否 |
> | sites / slots / networkConfig | 否 |
> | sourceControls | 否 |
> | staticSites | 是 |
> | validate | 否 |
> | verifyHostingEnvironmentVnet | 否 |

<!--Not Available on ## Microsoft.WindowsDefenderATP-->
<!-- Not Available on ## Microsoft.WindowsESU-->
<!--Not Available on ## Microsoft.WindowsIoT-->
<!-- Not Available on ## Microsoft.WorkloadBuilder-->
<!--Not Available on ## Microsoft.WorkloadMonitor-->
<!--Not Available on [complete-mode-deletion.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/complete-mode-deletion.csv)-->

<!-- Update_Description: update meta properties, wording update, update link -->