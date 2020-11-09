---
title: 标注策略 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的标注策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 04/01/2020
ms.date: 07/01/2020
ms.openlocfilehash: 28f0e6c0415f18a0c3e88f34e5ce42ee1150c6a0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106015"
---
# <a name="callout-policy"></a>标注策略

Azure 数据资源管理器群集可以在许多不同的方案中与外部服务通信。
群集管理员可以通过更新群集的标注策略来管理用于外部调用的授权域。

标注策略在群集级别进行管理，分为以下类型。
* `kusto` - 控制 Azure 数据资源管理器跨群集查询。
* `sql` - 控制 [SQL 插件](../query/sqlrequestplugin.md)。
* `cosmosdb` - 控制 [CosmosDB 插件](../query/cosmosdb-plugin.md)。
* `sandbox_artifacts` - 控制沙盒插件 ([python](../query/pythonplugin.md) | [R](../query/rplugin.md))。
* `external_data` - 通过[外部表](../query/schema-entities/externaltables.md)或 [externaldata](../query/externaldata-operator.md) 运算符控制对外部数据的访问。

标注策略由以下内容组成。

* **CalloutType** - 定义标注类型，可以为 `kusto` 或 `sql`。
* **CalloutUriRegex** - 指定标注域允许的正则表达式
* **CanCall** - 指示是否允许外部调用标注。

## <a name="predefined-callout-policies"></a>预定义标注策略

下表显示了一组预定义的标注策略，这些策略已在所有 Azure 数据资源管理器群集上进行了预配置，以使标注能够选择服务。

|服务      |云        |指定用途  |允许的域 |
|-------------|-------------|-------------|-------------|
|Kusto |`Public Azure` |跨群集查询 |`^[^.]*\.kusto\.chinacloudapi\.cn$` <br> `^[^.]*\.kustomfa\.windows\.net$` |
|Kusto |`Black Forest` |跨群集查询 |`^[^.]*\.kusto\.cloudapi\.de$` <br> `^[^.]*\.kustomfa\.cloudapi\.de$` |
|Kusto |`Fairfax` |跨群集查询 |`^[^.]*\.kusto\.usgovcloudapi\.net$` <br> `^[^.]*\.kustomfa\.usgovcloudapi\.net$` |
|Kusto |`Mooncake` |跨群集查询 |`^[^.]*\.kusto\.chinacloudapi\.cn$` <br> `^[^.]*\.kustomfa\.chinacloudapi\.cn$` |
|Azure DB |`Public Azure` |SQL 请求 |`^[^.]*\.database\.chinacloudapi\.cn$` <br> `^[^.]*\.databasemfa\.windows\.net$` |
|Azure DB |`Black Forest` |SQL 请求 |`^[^.]*\.database\.cloudapi\.de$` <br> `^[^.]*\.databasemfa\.cloudapi\.de$` |
|Azure DB |`Fairfax` |SQL 请求 |`^[^.]*\.database\.usgovcloudapi\.net$` <br> `^[^.]*\.databasemfa\.usgovcloudapi\.net$` |
|Azure DB |`Mooncake` |SQL 请求 |`^[^.]*\.database\.chinacloudapi\.cn$` <br> `^[^.]*\.databasemfa\.chinacloudapi\.cn$` |
|基线服务 |公共 Azure |基线请求 |`baseliningsvc-int.azurewebsites.net` <br> `baseliningsvc-ppe.azurewebsites.net` <br> `baseliningsvc-prod.azurewebsites.net` |

## <a name="control-commands"></a>控制命令

命令需要 [AllDatabasesAdmin](access-control/role-based-authorization.md) 权限。

**显示所有已配置的标注策略**

```kusto
.show cluster policy callout
```

**更改标注策略**

```kusto
.alter cluster policy callout @'[{"CalloutType": "sql","CalloutUriRegex": "sqlname.database.chinacloudapi.cn","CanCall": true}]'
```

**添加一组允许的标注**

```kusto
.alter-merge cluster policy callout @'[{"CalloutType": "sql","CalloutUriRegex": "sqlname.database.chinacloudapi.cn","CanCall": true}]'
```

**删除所有非不可变的标注策略**

```kusto
.delete cluster policy callout
```
