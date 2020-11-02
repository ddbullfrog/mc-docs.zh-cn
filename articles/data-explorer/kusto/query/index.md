---
title: 概述 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的概述。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/07/2019
ms.date: 10/29/2020
ms.openlocfilehash: 064a8b51d8f8295f3dd63e07cca5756985c4841c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104789"
---
# <a name="overview"></a>概述

Kusto 查询是处理数据并返回结果的只读请求。
该请求用纯文本形式表示，使用的数据流模型旨在使语法更易于阅读、创作和自动执行。 该查询使用以类似于 SQL 的层次结构组织的架构实体：数据库、表和列。

该查询包含一系列由分号 (`;`) 分隔的查询语句，至少有一个语句是[表格表达式语句](tabularexpressionstatements.md)，该语句可以生成按列和行的表式网格排列的数据。 查询的表格表达式语句生成查询结果。

表格表达语句的语法具有从一个表格查询运算符到另一个表格查询运算符的表格数据流，从数据源（例如，数据库中的一个表，或产生数据的运算符）开始，然后流经一组数据转换运算符，这些运算符通过使用竖线 (`|`) 分隔符绑定在一起。

例如，下面的 Kusto 查询有一个单独的语句，它是一个表格表达式语句。 该语句从对名为 `StormEvents` 的表引用开始（宿主此表的数据库在此是隐式的，并且是连接信息的一部分）。 然后，该表的数据（行）依次由 `StartTime` 列的值和 `State` 列的值进行筛选。 然后查询返回“幸存”的行计数。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents 
| where StartTime >= datetime(2007-11-01) and StartTime < datetime(2007-12-01)
| where State == "FLORIDA"  
| count 
```

若要运行此查询，请[单击此处](https://dataexplorer.azure.cn/clusters/help/databases/Samples?query=H4sIAAAAAAAAAwsuyS/KdS1LzSspVuDlqlEoz0gtSlUILkksKgnJzE1VsLNVSEksSS0BsjWMDAzMdQ0NdQ0MNRUS81KQVNmgKzICKUIxryRVwdZWQcnNxz/I08VRSQFsW3J+aV6JAgAwMx4+hAAAAA==)。
在此情况下，结果将为：

|Count|
|-----|
|   23|
