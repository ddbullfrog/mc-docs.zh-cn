---
title: 查询语句 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的查询语句。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 12/10/2019
ms.date: 09/30/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 30beb46e9e6af71cd25a646dd8e7ac73bc6b5386
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105108"
---
# <a name="query-statement-types"></a>查询语句类型

::: zone pivot="azuredataexplorer"

查询包含一个或多个“查询语句”，用分号 (`;`) 分隔。
其中至少有一个查询语句必须是[表格表达式语句](./tabularexpressionstatements.md)。
表格表达式语句会生成一个或多个表格结果。
当查询包含多个表格表达式语句时，查询将有一[批](./batches.md)表格表达式语句，且这些语句生成的表格结果都由查询返回。

两种类型的查询语句：

* 主要由用户使用的语句（[用户查询语句](#user-query-statements)），
* 为支持这样的场景而设计的语句：中间层应用程序接受用户查询并将修改后的版本发送到 Kusto（[应用程序查询语句](#application-query-statements)）。

在这两种情况下，某些查询语句非常有用。

> [!NOTE]
> 查询语句的“效果”从该语句出现在查询中时开始，到查询结束时结束。 查询完成后，将释放其所有资源，且查询不会影响以后的查询（除副作用外，比如将查询记录在所有查询的日志中或缓存查询结果。）

## <a name="user-query-statements"></a>用户查询语句

下面是用户查询语句的列表：

* [let 语句](./letstatement.md)定义名称和表达式之间的绑定。
  let 语句可用于将较长的查询分解为更易于理解的小型命名部分。

* [set 语句](./setstatement.md)设置查询选项，该选项会影响查询的处理方式及其返回的结果。

* [ 语句](./tabularexpressionstatements.md)是最重要的查询语句，会返回“有趣的”数据作为结果。

## <a name="application-query-statements"></a>应用程序查询语句

以下是用户查询语句的列表：

* [alias 语句](./aliasstatement.md) 定义（同一群集或远程群集上）的另一个数据库的别名。

* [pattern 语句](./patternstatement.md)可由基于 Kusto 构建的应用程序用来向其用户公开查询语言以将其自身注入查询名称解析过程。

* [语句](./queryparametersstatement.md)由基于 Kusto 构建的应用程序用来防范注入式攻击（原理类似于命令参数保护 SQL 免于 SQL 注入攻击。）

* [restrict 语句](./restrictstatement.md)由在 Kusto 的顶部构建的应用程序用来将查询限制于 Kusto 中的特定数据子集（包括限制对特定列和记录的访问。）

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
