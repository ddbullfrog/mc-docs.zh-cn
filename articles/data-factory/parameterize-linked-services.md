---
title: 参数化 Azure 数据工厂中的链接服务
description: 了解如何参数化 Azure 数据工厂中的链接服务，并在运行时传递动态值。
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
origin.date: 09/21/2020
ms.date: 10/19/2020
author: WenJason
ms.author: v-jay
manager: digimobile
ms.openlocfilehash: d05aab5307045b6d98982e746736f52c47cd2197
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121720"
---
# <a name="parameterize-linked-services-in-azure-data-factory"></a>参数化 Azure 数据工厂中的链接服务

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

现在可以参数化链接服务并在运行时传递动态值。 例如，如果要连接到同一逻辑 SQL Server 上的不同数据库，则现在可以在链接服务定义中参数化数据库名称。 这样就不需为逻辑 SQL Server 上的每个数据库创建链接服务。 也可以参数化链接服务定义中的其他属性 - 例如，用户名。

可以使用 Azure 门户中的数据工厂 UI 或编程接口来参数化链接服务。

> [!TIP]
> 我们建议不要参数化密码或机密。 而应将所有连接字符串都存储在 Azure Key Vault 中，并参数化 *机密名称* 。

## <a name="supported-data-stores"></a>支持的数据存储

可以将任何类型的链接服务参数化。
在 UI 上创作链接服务时，数据工厂会为以下类型的连接器提供内置参数化体验。 在“链接服务创建/编辑”边栏选项卡中，可以找到新参数的选项并添加动态内容。

- Amazon Redshift
- Azure Cosmos DB (SQL API)
- Azure Database for MySQL
- Azure SQL 数据库
- Azure Synapse Analytics（以前称为 SQL DW）
- MySQL
- Oracle
- SQL Server
- 泛型 HTTP
- 泛型 REST

对于其他类型，可以通过在 UI 上编辑 JSON 来参数化链接服务：

- 在“链接服务创建/编辑”边栏选项卡中 -> 展开底部的“高级”-> 选中“以 JSON 格式指定动态内容”复选框 -> 指定链接服务 JSON 有效负载。 
- 或者，在创建没有参数化的链接服务后，转到[管理中心](author-visually.md#management-hub) ->“链接服务”-> 查找特定的链接服务 -> 单击“代码”（“{}”按钮）以编辑 JSON。 

请参考 [JSON 示例](#json)来添加 ` parameters` 节，以定义参数并使用 ` @{linkedService().paraName} ` 来引用参数。

## <a name="data-factory-ui"></a>数据工厂 UI

![将动态内容添加到“链接服务”定义](media/parameterize-linked-services/parameterize-linked-services-image1.png)

![新建参数](media/parameterize-linked-services/parameterize-linked-services-image2.png)

## <a name="json"></a>JSON

```json
{
    "name": "AzureSqlDatabase",
    "properties": {
        "type": "AzureSqlDatabase",
        "typeProperties": {
            "connectionString": "Server=tcp:myserver.database.chinacloudapi.cn,1433;Database=@{linkedService().DBName};User ID=user;Password=fake;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        },
        "connectVia": null,
        "parameters": {
            "DBName": {
                "type": "String"
            }
        }
    }
}
```
