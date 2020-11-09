---
title: 适用于 Azure Cosmos DB 表 API 的资源管理器模板
description: 使用 Azure 资源管理器模板创建和配置 Azure Cosmos DB 表 API。
ms.service: cosmos-db
ms.topic: how-to
origin.date: 05/19/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: yes|no
ms.testdate: 11/09/2020null
ms.author: v-yeche
ms.openlocfilehash: fd89a93c6101c2a02cd1354c8cc31af25b8493e9
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94329147"
---
# <a name="manage-azure-cosmos-db-table-api-resources-using-azure-resource-manager-templates"></a>使用 Azure 资源管理器模板管理 Azure Cosmos DB 表 API 资源
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

在本文中，你将了解如何使用 Azure 资源管理器模板来帮助部署和管理 Azure Cosmos DB 帐户、数据库和容器。

本文仅提供表 API 帐户的示例，若要查找其他 API 类型帐户的示例，请参阅以下文章：将 Azure 资源管理器模板与适用于 [Cassandra](./templates-samples-cassandra.md)、[Gremlin](./templates-samples-gremlin.md)、[MongoDB](./templates-samples-mongodb.md)、[SQL](./manage-with-templates.md)的 Azure Cosmos DB API 配合使用。

> [!IMPORTANT]
>
> * 帐户名称限制为 44 个字符，全部小写。
> * 若要更改吞吐量值，请用更新的 RU/s 重新部署模板。
> * 当你在 Azure Cosmos 帐户中添加或删除位置时，无法同时修改其他属性。 必须单独执行这些操作。

若要创建以下任何 Azure Cosmos DB 资源，请将下列示例模板复制到新的 json 文件中。 在部署具有不同名称和值的同一资源的多个实例时，可以选择创建要使用的参数 json 文件。 可以通过多种方式部署 Azure 资源管理器模板，包括 [Azure 门户](../azure-resource-manager/templates/deploy-portal.md)、[Azure CLI](../azure-resource-manager/templates/deploy-cli.md)、[Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md) 和 [GitHub](../azure-resource-manager/templates/deploy-to-azure-button.md)。

> [!TIP]
> 若要在使用表 API 时启用共享吞吐量，请在 Azure 门户中启用帐户级吞吐量。

<a name="create-autoscale"></a>

## <a name="azure-cosmos-account-for-table-with-autoscale-throughput"></a>具有自动缩放吞吐量的表的 Azure Cosmos 帐户

此模板将为包含一个具有自动缩放吞吐量的表的表 API 创建一个 Azure Cosmos 帐户。 此模板还支持从 Azure 快速入门模板库进行一键部署。

[!INCLUDE [azure-raw-githubusercontent-azurechinacloud-environment-notice](../../includes/azure-raw-githubusercontent-azurechinacloud-environment-notice.md)]

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="部署到 Azure":::](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cosmosdb-table-autoscale%2Fazuredeploy.json)

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
       "accountName": {
          "type": "string",
          "defaultValue": "[concat('table-', uniqueString(resourceGroup().id))]",
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
       "primaryRegion":{
          "type":"string",
          "metadata": {
             "description": "The primary replica region for the Cosmos DB account."
          }
       },
       "secondaryRegion":{
          "type":"string",
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
             "description": "Enable automatic failover for regions. Ignored when Multi-Master is enabled"
          }
       },
       "tableName": {
          "type": "string",
          "metadata": {
             "description": "The name for the table"
          }
       },
        "autoscaleMaxThroughput": {
            "type": "int",
            "defaultValue": 4000,
            "minValue": 4000,
            "maxValue": 1000000,
            "metadata": {
                "description": "Maximum autoscale throughput for the table"
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
                  "name": "EnableTable" 
               }
            ],
            "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
            "locations": "[variables('locations')]",
            "databaseAccountOfferType": "Standard",
            "enableAutomaticFailover": "[parameters('automaticFailover')]"
            }
       },
       {
          "type": "Microsoft.DocumentDB/databaseAccounts/tables",
          "name": "[concat(variables('accountName'), '/', parameters('tableName'))]",
          "apiVersion": "2020-04-01",
          "dependsOn": [ 
             "[resourceId('Microsoft.DocumentDB/databaseAccounts/', variables('accountName'))]" 
            ],
          "properties":{
             "resource":{
                "id": "[parameters('tableName')]"
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

## <a name="azure-cosmos-account-for-table-with-standard-provisioned-throughput"></a>具有标准预配吞吐量的表的 Azure Cosmos 帐户

此模板将为包含一个具有标准吞吐量的表的表 API 创建 Azure Cosmos 帐户。 此模板还支持从 Azure 快速入门模板库进行一键部署。

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="部署到 Azure":::](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cosmosdb-table%2Fazuredeploy.json)

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string",
            "defaultValue": "[concat('table-', uniqueString(resourceGroup().id))]",
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
        "primaryRegion":{
            "type":"string",
            "metadata": {
                "description": "The primary replica region for the Cosmos DB account."
            }
        },
        "secondaryRegion":{
            "type":"string",
            "metadata": {
              "description": "The secondary replica region for the Cosmos DB account."
          }
        },
        "defaultConsistencyLevel": {
            "type": "string",
            "defaultValue": "Session",
            "allowedValues": [ "Eventual", "ConsistentPrefix", "Session", "BoundedStaleness", "Strong" ],
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
            "allowedValues": [ true, false ],
            "metadata": {
                "description": "Enable automatic failover for regions. Ignored when Multi-Master is enabled"
            }
        },
        "tableName": {
            "type": "string",
            "metadata": {
                "description": "The name for the table"
            }
        },
        "throughput": {
            "type": "int",
            "defaultValue": 400,
            "minValue": 400,
            "maxValue": 1000000,
            "metadata": {
                "description": "The throughput for the table"
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
        "locations": 
        [ 
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
    "resources": 
    [
        {
            "type": "Microsoft.DocumentDB/databaseAccounts",
            "name": "[variables('accountName')]",
            "apiVersion": "2020-03-01",
            "location": "[parameters('location')]",
            "kind": "GlobalDocumentDB",
            "properties": {
                "capabilities": [{ "name": "EnableTable" }],
                "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
                "locations": "[variables('locations')]",
                "databaseAccountOfferType": "Standard",
                "enableAutomaticFailover": "[parameters('automaticFailover')]"
            }
        },
        {
            "type": "Microsoft.DocumentDB/databaseAccounts/tables",
            "name": "[concat(variables('accountName'), '/', parameters('tableName'))]",
            "apiVersion": "2020-03-01",
            "dependsOn": [ "[resourceId('Microsoft.DocumentDB/databaseAccounts/', variables('accountName'))]" ],
            "properties":{
                "resource":{
                    "id": "[parameters('tableName')]"
                },
                "options": { "throughput": "[parameters('throughput')]" }
            }
        }
    ]
}
```

## <a name="next-steps"></a>后续步骤

下面是一些其他资源：

* [Azure 资源管理器文档](../azure-resource-manager/index.yml)
* [Azure Cosmos DB 资源提供程序架构](https://docs.microsoft.com/azure/templates/microsoft.documentdb/allversions)
* [Azure Cosmos DB 快速入门模板](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.DocumentDB&pageNumber=1&sort=Popular)
* [排查常见 Azure 资源管理器部署错误](../azure-resource-manager/templates/common-deployment-errors.md)

<!-- Update_Description: new article about templates samples table -->
<!--NEW.date: 11/09/2020-->