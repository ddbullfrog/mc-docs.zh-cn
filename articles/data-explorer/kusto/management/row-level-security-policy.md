---
title: RowLevelSecurity 策略 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 RowLevelSecurity 策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 10/11/2020
ms.date: 10/29/2020
ms.openlocfilehash: 393179006350878a2a7fd165bc7dc97967fdde14
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104277"
---
# <a name="row_level_security-policy-command"></a>row_level_security 策略命令

本文介绍用于为数据库表配置 [row_level_security 策略](rowlevelsecuritypolicy.md)的命令。

## <a name="displaying-the-policy"></a>显示策略

若要显示策略，请使用以下命令：

```kusto
.show table <table_name> policy row_level_security
```

## <a name="configuring-the-policy"></a>配置策略

对表启用 row_level_security 策略：

```kusto
.alter table <table_name> policy row_level_security enable "<query>"
```

对表禁用 row_level_security 策略：

```kusto
.alter table <table_name> policy row_level_security disable "<query>"
```

即使策略被禁用，也可以强制将其应用于单个查询。 在查询前输入以下行：

`set query_force_row_level_security;`

如果想尝试各种符合 row_level_security 策略的查询，但又不希望它对用户产生实际影响，那么这将非常有用。 对查询有信心后，就启用该策略。

> [!NOTE]
> 以下限制适用于 `query`：
>
> * 查询应生成与在其上定义该策略的表完全相同的架构。 也就是说，查询的结果应该返回与原始表完全相同的列（顺序相同且名称和类型也相同）。
> * 查询只能使用以下运算符：`extend`、`where`、`project`、`project-away`、`project-rename`、`project-reorder`、`join` 和 `union`。
> * 查询无法引用在其上启用了 RLS 的其他表。
> * 查询可以是以下任一项，也可以是它们的组合：
>    * 查询（例如，`<table_name> | extend CreditCardNumber = "****"`）
>    * 函数（例如，`AnonymizeSensitiveData`）
>    * 数据表（例如，`datatable(Col1:datetime, Col2:string) [...]`）

> [!TIP]
> 以下函数通常用于 row_level_security 查询：
> * [current_principal()](../query/current-principalfunction.md)
> * [current_principal_details()](../query/current-principal-detailsfunction.md)
> * [current_principal_is_member_of()](../query/current-principal-ismemberoffunction.md)

**示例**

```kusto
.create-or-alter function with () TrimCreditCardNumbers() {
    let UserCanSeeFullNumbers = current_principal_is_member_of('aadgroup=super_group@domain.com');
    let AllData = Customers | where UserCanSeeFullNumbers;
    let PartialData = Customers | where not(UserCanSeeFullNumbers) | extend CreditCardNumber = "****";
    union AllData, PartialData
}

.alter table Customers policy row_level_security enable "TrimCreditCardNumbers"
```

**性能说明** ：首先评估 `UserCanSeeFullNumbers`，然后评估 `AllData` 或 `PartialData`，而不是同时评估这两者，这是预期的结果。
可在[此处](rowlevelsecuritypolicy.md#performance-impact-on-queries)详细了解 RLS 对性能的影响。

## <a name="deleting-the-policy"></a>删除策略

```kusto
.delete table <table_name> policy row_level_security
```
