---
title: 适用于 Azure Cosmos DB Gremlin API 的资源管理器模板
description: 使用 Azure 资源管理器模板创建和配置 Azure Cosmos DB Gremlin API。
ms.service: cosmos-db
ms.topic: how-to
origin.date: 10/14/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: afd29d510ad5f6e2676772b4164466f50a478fb7
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94329151"
---
<!--Verified successfully-->
# <a name="manage-azure-cosmos-db-gremlin-api-resources-using-azure-resource-manager-templates"></a>使用 Azure 资源管理器模板管理 Azure Cosmos DB Gremlin API 资源
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

在本文中，你将了解如何使用 Azure 资源管理器模板来帮助部署和管理 Azure Cosmos DB 帐户、数据库和图。

本文仅提供 Gremlin API 帐户的示例。若要查找其他 API 类型帐户的示例，请参阅以下文章：将 Azure 资源管理器模板与适用于 [Cassandra](templates-samples-cassandra.md)、[SQL](templates-samples-sql.md)、[MongoDB](templates-samples-mongodb.md)、[表](templates-samples-table.md)的 Azure Cosmos DB API 配合使用。

> [!IMPORTANT]
>
> * 帐户名称限制为 44 个字符，全部小写。
> * 若要更改吞吐量值，请用更新的 RU/s 重新部署模板。
> * 当你在 Azure Cosmos 帐户中添加或删除位置时，无法同时修改其他属性。 必须单独执行这些操作。

若要创建以下任何 Azure Cosmos DB 资源，请将下列示例模板复制到新的 json 文件中。 在部署具有不同名称和值的同一资源的多个实例时，可以选择创建要使用的参数 json 文件。 可以通过多种方式部署 Azure 资源管理器模板，包括 [Azure 门户](../azure-resource-manager/templates/deploy-portal.md)、[Azure CLI](../azure-resource-manager/templates/deploy-cli.md)、[Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md) 和 [GitHub](../azure-resource-manager/templates/deploy-to-azure-button.md)。

<a name="create-autoscale"></a>

## <a name="azure-cosmos-db-account-for-gremlin-with-autoscale-provisioned-throughput"></a>预配了自动缩放吞吐量的 Gremlin 的 Azure Cosmos DB 帐户

此模板将为 Gremlin API（具有自动缩放吞吐量的数据库和图）创建一个 Azure Cosmos 帐户。 此模板还支持从 Azure 快速入门模板库进行一键部署。

[!INCLUDE [azure-raw-githubusercontent-azurechinacloud-environment-notice](../../includes/azure-raw-githubusercontent-azurechinacloud-environment-notice.md)]

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="部署到 Azure":::](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cosmosdb-gremlin-autoscale%2Fazuredeploy.json)

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string",
            "defaultValue": "[concat('gremlin-', uniqueString(resourceGroup().id))]",
            "metadata": {
                "description": "Cosmos DB account name"
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for the Cosmos DB account."
            }
        },
        "primaryRegion": {
            "type": "string",
            "metadata": {
                "description": "The primary replica region for the Cosmos DB account."
            }
        },
        "secondaryRegion": {
            "type": "string",
            "metadata": {
                "description": "The secondary replica region for the Cosmos DB account."
            }
        },
        "defaultConsistencyLevel": {
            "type": "string",
            "defaultValue": "Session",
            "allowedValues": [
                "Eventual",
                "ConsistentPrefix",
                "Session",
                "BoundedStaleness",
                "Strong"
            ],
            "metadata": {
                "description": "The default consistency level of the Cosmos DB account."
            }
        },
        "maxStalenessPrefix": {
            "type": "int",
            "defaultValue": 100000,
            "minValue": 10,
            "maxValue": 2147483647,
            "metadata": {
                "description": "Max stale requests. Required for BoundedStaleness. Valid ranges, Single Region: 10 to 1000000. Multi Region: 100000 to 1000000."
            }
        },
        "maxIntervalInSeconds": {
            "type": "int",
            "defaultValue": 300,
            "minValue": 5,
            "maxValue": 86400,
            "metadata": {
                "description": "Max lag time (seconds). Required for BoundedStaleness. Valid ranges, Single Region: 5 to 84600. Multi Region: 300 to 86400."
            }
        },
        "automaticFailover": {
            "type": "bool",
            "defaultValue": true,
            "allowedValues": [
                true,
                false
            ],
            "metadata": {
                "description": "Enable automatic failover for regions"
            }
        },
        "databaseName": {
            "type": "string",
            "metadata": {
                "description": "The name for the Gremlin database"
            }
        },
        "graphName": {
            "type": "string",
            "metadata": {
                "description": "The name for the Gremlin graph"
            }
        },
        "autoscaleMaxThroughput": {
            "type": "int",
            "defaultValue": 4000,
            "minValue": 4000,
            "maxValue": 1000000,
            "metadata": {
                "description": "Maximum autoscale throughput for the graph"
            }
        }
    },
    "variables": {
        "accountName": "[toLower(parameters('accountName'))]",
        "consistencyPolicy": {
            "Eventual": {
                "defaultConsistencyLevel": "Eventual"
            },
            "ConsistentPrefix": {
                "defaultConsistencyLevel": "ConsistentPrefix"
            },
            "Session": {
                "defaultConsistencyLevel": "Session"
            },
            "BoundedStaleness": {
                "defaultConsistencyLevel": "BoundedStaleness",
                "maxStalenessPrefix": "[parameters('maxStalenessPrefix')]",
                "maxIntervalInSeconds": "[parameters('maxIntervalInSeconds')]"
            },
            "Strong": {
                "defaultConsistencyLevel": "Strong"
            }
        },
        "locations": [
            {
                "locationName": "[parameters('primaryRegion')]",
                "failoverPriority": 0,
                "isZoneRedundant": false
            },
            {
                "locationName": "[parameters('secondaryRegion')]",
                "failoverPriority": 1,
                "isZoneRedundant": false
            }
        ]
    },
    "resources": [
        {
            "type": "Microsoft.DocumentDB/databaseAccounts",
            "name": "[variables('accountName')]",
            "apiVersion": "2020-04-01",
            "location": "[parameters('location')]",
            "kind": "GlobalDocumentDB",
            "properties": {
                "capabilities": [
                    {
                        "name": "EnableGremlin"
                    }
                ],
                "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
                "locations": "[variables('locations')]",
                "databaseAccountOfferType": "Standard",
                "enableAutomaticFailover": "[parameters('automaticFailover')]"
            }
        },
        {
            "type": "Microsoft.DocumentDB/databaseAccounts/gremlinDatabases",
            "name": "[concat(variables('accountName'), '/', parameters('databaseName'))]",
            "apiVersion": "2020-04-01",
            "dependsOn": [
                "[resourceId('Microsoft.DocumentDB/databaseAccounts/', variables('accountName'))]"
            ],
            "properties": {
                "resource": {
                    "id": "[parameters('databaseName')]"
                }
            }
        },
        {
            "type": "Microsoft.DocumentDb/databaseAccounts/gremlinDatabases/graphs",
            "name": "[concat(variables('accountName'), '/', parameters('databaseName'), '/', parameters('graphName'))]",
            "apiVersion": "2020-04-01",
            "dependsOn": [
                "[resourceId('Microsoft.DocumentDB/databaseAccounts/gremlinDatabases', variables('accountName'),  parameters('databaseName'))]"
            ],
            "properties": {
                "resource": {
                    "id": "[parameters('graphName')]",
                    "indexingPolicy": {
                        "indexingMode": "consistent",
                        "includedPaths": [
                            {
                                "path": "/*"
                            }
                        ],
                        "excludedPaths": [
                            {
                                "path": "/myPathToNotIndex/*"
                            }
                        ]
                    },
                    "partitionKey": {
                        "paths": [
                            "/myPartitionKey"
                        ],
                        "kind": "Hash"
                    }
                },
                "options": {
                    "autoscaleSettings": { 
                        "maxThroughput": "[parameters('autoscaleMaxThroughput')]" 
                    }
                }
            }
        }
    ]
}
```

<a name="create-manual"></a>

## <a name="azure-cosmos-db-account-for-gremlin-with-standard-provisioned-throughput"></a>预配了标准吞吐量的 Gremlin 的 Azure Cosmos DB 帐户

此模板将为 Gremlin API（具有标准手动吞吐量的数据库和图）创建一个 Azure Cosmos 帐户。 此模板还支持从 Azure 快速入门模板库进行一键部署。

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="部署到 Azure":::](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cosmosdb-gremlin%2Fazuredeploy.json)

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string",
            "defaultValue": "[uniqueString(resourceGroup().id)]",
            "metadata": {
                "description": "Cosmos DB account name"
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for the Cosmos DB account."
            }
        },
        "primaryRegion": {
            "type": "string",
            "metadata": {
                "description": "The primary replica region for the Cosmos DB account."
            }
        },
        "secondaryRegion": {
            "type": "string",
            "metadata": {
                "description": "The secondary replica region for the Cosmos DB account."
            }
        },
        "defaultConsistencyLevel": {
            "type": "string",
            "defaultValue": "Session",
            "allowedValues": [
                "Eventual",
                "ConsistentPrefix",
                "Session",
                "BoundedStaleness",
                "Strong"
            ],
            "metadata": {
                "description": "The default consistency level of the Cosmos DB account."
            }
        },
        "maxStalenessPrefix": {
            "type": "int",
            "defaultValue": 100000,
            "minValue": 10,
            "maxValue": 2147483647,
            "metadata": {
                "description": "Max stale requests. Required for BoundedStaleness. Valid ranges, Single Region: 10 to 1000000. Multi Region: 100000 to 1000000."
            }
        },
        "maxIntervalInSeconds": {
            "type": "int",
            "defaultValue": 300,
            "minValue": 5,
            "maxValue": 86400,
            "metadata": {
                "description": "Max lag time (seconds). Required for BoundedStaleness. Valid ranges, Single Region: 5 to 84600. Multi Region: 300 to 86400."
            }
        },
        "automaticFailover": {
            "type": "bool",
            "defaultValue": true,
            "allowedValues": [
                true,
                false
            ],
            "metadata": {
                "description": "Enable automatic failover for regions"
            }
        },
        "databaseName": {
            "type": "string",
            "defaultValue": "database1",
            "metadata": {
                "description": "The name for the Gremlin database"
            }
        },
        "graphName": {
            "type": "string",
            "defaultValue": "graph1",
            "metadata": {
                "description": "The name for the Gremlin graph"
            }
        },
        "throughput": {
            "type": "int",
            "defaultValue": 400,
            "minValue": 400,
            "maxValue": 1000000,
            "metadata": {
                "description": "Throughput for the Gremlin graph"
            }
        }
    },
    "variables": {
        "accountName": "[toLower(parameters('accountName'))]",
        "consistencyPolicy": {
            "Eventual": {
                "defaultConsistencyLevel": "Eventual"
            },
            "ConsistentPrefix": {
                "defaultConsistencyLevel": "ConsistentPrefix"
            },
            "Session": {
                "defaultConsistencyLevel": "Session"
            },
            "BoundedStaleness": {
                "defaultConsistencyLevel": "BoundedStaleness",
                "maxStalenessPrefix": "[parameters('maxStalenessPrefix')]",
                "maxIntervalInSeconds": "[parameters('maxIntervalInSeconds')]"
            },
            "Strong": {
                "defaultConsistencyLevel": "Strong"
            }
        },
        "locations": [
            {
                "locationName": "[parameters('primaryRegion')]",
                "failoverPriority": 0,
                "isZoneRedundant": false
            },
            {
                "locationName": "[parameters('secondaryRegion')]",
                "failoverPriority": 1,
                "isZoneRedundant": false
            }
        ]
    },
    "resources": [
        {
            "type": "Microsoft.DocumentDB/databaseAccounts",
            "name": "[variables('accountName')]",
            "apiVersion": "2020-03-01",
            "location": "[parameters('location')]",
            "kind": "GlobalDocumentDB",
            "properties": {
                "capabilities": [
                    {
                        "name": "EnableGremlin"
                    }
                ],
                "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
                "locations": "[variables('locations')]",
                "databaseAccountOfferType": "Standard",
                "enableAutomaticFailover": "[parameters('automaticFailover')]"
            }
        },
        {
            "type": "Microsoft.DocumentDB/databaseAccounts/gremlinDatabases",
            "name": "[concat(variables('accountName'), '/', parameters('databaseName'))]",
            "apiVersion": "2020-03-01",
            "dependsOn": [
                "[resourceId('Microsoft.DocumentDB/databaseAccounts/', variables('accountName'))]"
            ],
            "properties": {
                "resource": {
                    "id": "[parameters('databaseName')]"
                }
            }
        },
        {
            "type": "Microsoft.DocumentDb/databaseAccounts/gremlinDatabases/graphs",
            "name": "[concat(variables('accountName'), '/', parameters('databaseName'), '/', parameters('graphName'))]",
            "apiVersion": "2020-03-01",
            "dependsOn": [
                "[resourceId('Microsoft.DocumentDB/databaseAccounts/gremlinDatabases', variables('accountName'),  parameters('databaseName'))]"
            ],
            "properties": {
                "resource": {
                    "id": "[parameters('graphName')]",
                    "indexingPolicy": {
                        "indexingMode": "consistent",
                        "includedPaths": [
                            {
                                "path": "/*"
                            }
                        ],
                        "excludedPaths": [
                            {
                                "path": "/myPathToNotIndex/*"
                            }
                        ]
                    },
                    "partitionKey": {
                        "paths": [
                            "/myPartitionKey"
                        ],
                        "kind": "Hash"
                    },
                    "options": {
                        "throughput": "[parameters('throughput')]"
                    }
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>后续步骤

下面是一些其他资源：

* [Azure 资源管理器文档](../azure-resource-manager/index.yml)

<!--Not Available on  - [Azure Cosmos DB resource provider schema](https://docs.microsoft.com/azure/templates/microsoft.documentdb/allversions)-->

* [Azure Cosmos DB 快速入门模板](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.DocumentDB&pageNumber=1&sort=Popular)
* [排查常见 Azure 资源管理器部署错误](../azure-resource-manager/templates/common-deployment-errors.md)

<!-- Update_Description: new article about templates samples gremlin -->
<!--NEW.date: 11/09/2020-->