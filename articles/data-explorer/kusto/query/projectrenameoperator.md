---
title: project-rename 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 project-rename 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 14fde3f2e6ba4dbd5fa283f3e51d71af90d16f89
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104192"
---
# <a name="project-rename-operator"></a>project-rename 运算符

对结果输出中的列重命名。

```kusto
T | project-rename new_column_name = column_name
```

## <a name="syntax"></a>语法

*T* `| project-rename` *NewColumnName* = *ExistingColumnName* [`,` ...]

## <a name="arguments"></a>参数

* *T* ：输入表。
* NewColumnName：列的新名称。 
* ExistingColumnName：列的现有名称。 

## <a name="returns"></a>返回

一个表，其列的顺序与现有表中的顺序相同，但列已重命名。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print a='a', b='b', c='c'
|  project-rename new_b=b, new_a=a
```

|new_a|new_b|c|
|---|---|---|
|a|b|c|
