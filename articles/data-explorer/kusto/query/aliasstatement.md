---
title: Alias 语句 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 Alias 语句。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 0585e5dc54731074412c778749a6df86fc306ee4
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105055"
---
# <a name="alias-statement"></a>Alias 语句

::: zone pivot="azuredataexplorer"

使用 Alias 语句可为数据库定义别名，之后可在同一查询中使用该别名。

当你要处理多个群集，但希望看起来像是在处理较少的群集时，这很有用。
必须根据以下语法定义别名，其中 clustername 和 databasename 是现有的有效实体 。

## <a name="syntax"></a>语法

`alias` database['DatabaseAliasName'] `=` cluster("https://clustername.kusto.chinacloudapi.cn:443").database("databasename")  

`alias` database DatabaseAliasName `=` cluster("https://clustername.kusto.chinacloudapi.cn:443").database("databasename")  

* 'DatabaseAliasName' 可以是现有名称，也可以是新名称。
* 映射的 cluster-uri 和映射的 database-name 必须出现在双引号 (") 或单引号 (') 内

## <a name="examples"></a>示例

```kusto
alias database["wiki"] = cluster("https://somecluster.kusto.chinacloudapi.cn:443").database("somedatabase");
database("wiki").PageViews | count 
```

```kusto
alias database Logs = cluster("https://othercluster.kusto.chinacloudapi.cn:443").database("otherdatabase");
database("Logs").Traces | count 
```

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
