---
title: Kusto 保留策略控制删除数据的方式 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的保留策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2020
ms.date: 10/29/2020
ms.openlocfilehash: f1639ef4e0f0b6ac30aed3a97a64e1b61dc5e0af
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104278"
---
# <a name="retention-policy"></a>保留策略

保留策略控制自动从表或[具体化视图](materialized-views/materialized-view-overview.md)中删除数据的机制。 删除持续传递到表中且相关性基于期限的数据是非常有用的。 例如，该策略可用于保存诊断事件的表，这些事件在两周后可能变得无意义。

可以为特定的表或具体化视图，或整个数据库配置保留策略。 然后，该策略将应用于数据库中所有不会替代它的表。

设置保留策略对于不断引入数据的群集很重要，因为这会限制成本。

保留策略“外部”的数据可以删除。 不保证删除操作发生的具体时间。 即使触发了保留策略，数据也可能暂时保留。

保留策略经常设置为引入后限制数据的使用期限。 有关详细信息，请参阅 [SoftDeletePeriod](#the-policy-object)。

> [!NOTE]
> * 删除时间不确切。 系统保证在超过限制之前不会删除数据，但超过限制后不会立即删除数据。
> * 可以在表级保留策略中设置软删除周期 0，但不能在数据库级保留策略中进行此设置。
>   * 完成此操作后，引入的数据将不会提交到源表，从而无需长期保存数据。 因此，只能将 `Recoverability` 设置为 `Disabled`。
>   * 将数据引入到表中时，此类配置非常有用。
> 事务性[更新策略](updatepolicy.md)用于转换它，并将输出重定向到另一个表。

## <a name="the-policy-object"></a>策略对象

保留策略包括以下属性：

* **SoftDeletePeriod** ：
    * 保证数据可查询的时间跨度。 时间段从引入数据的时间开始度量。
    * 默认为 `100 years`。
    * 更改表或数据库的软删除时段时，新值将同时应用于现有数据和新数据。
* **可恢复性** ：
    * 删除数据后的数据可恢复性（启用/禁用）。
    * 默认为 `Enabled`。
    * 如果设置为 `Enabled`，则数据可以在软删除后的 14 天内被恢复。

## <a name="control-commands"></a>控制命令

* 使用 [.show 策略保留](../management/retention-policy.md)以显示数据库、表或[具体化视图](materialized-views/materialized-view-overview.md)的当前保留策略。
* 使用 [.alter 策略保留](../management/retention-policy.md)以更改数据库、表或[具体化视图](materialized-views/materialized-view-overview.md)的当前保留策略。

## <a name="defaults"></a>默认值

默认情况下，在创建数据库或表时，不定义保留策略。 通常，将创建数据库，然后立即由其创建者根据已知要求设置其保留策略。
对尚未设置策略的数据库或表的保留策略运行 [show 命令](../management/retention-policy.md)时，`Policy` 显示为 `null`。

可以使用以下命令应用默认的保留策略（具有上述默认值）。

```kusto
.alter database DatabaseName policy retention "{}"
.alter table TableName policy retention "{}"
.alter materialized-view ViewName policy retention "{}"
```

该命令会将以下策略对象应用于数据库或表。

```kusto
{
  "SoftDeletePeriod": "36500.00:00:00", "Recoverability":"Enabled"
}
```

可以使用以下命令来清除数据库或表的保留策略。

```kusto
.delete database DatabaseName policy retention
.delete table TableName policy retention
```

## <a name="examples"></a>示例

适用于有名为 `MyDatabase` 的数据库和有表 `MyTable1`、`MyTable2` 和 `MySpecialTable` 的群集。

### <a name="soft-delete-period-of-seven-days-and-recoverability-disabled"></a>软删除期为七天，禁用可恢复性

将数据库中的所有表的软删除期设置为七天，并禁用可恢复性。

* *选项 1（建议）* ：设置数据库级别的保留策略，并验证是否未设置表级别的策略。

```kusto
.delete table MyTable1 policy retention        // optional, only if the table previously had its policy set
.delete table MyTable2 policy retention        // optional, only if the table previously had its policy set
.delete table MySpecialTable policy retention  // optional, only if the table previously had its policy set
.alter-merge database MyDatabase policy retention softdelete = 7d recoverability = disabled
.alter-merge materialized-view ViewName policy retention softdelete = 7d 
```

* *选项 2* ：为每个表设置一个表级别的保留策略，其软删除期为七天，并禁用可恢复性。

```kusto
.alter-merge table MyTable1 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MyTable2 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MySpecialTable policy retention softdelete = 7d recoverability = disabled
```

### <a name="soft-delete-period-of-seven-days-and-recoverability-enabled"></a>软删除期为七天，启用可恢复性

* 将表 `MyTable1` 和 `MyTable2` 设置为具有七天的软删除期，并启用可恢复性。
* 将表 `MySpecialTable` 设置为具有 14 天的软删除期，并禁用可恢复性。

* *选项 1（建议）* ：设置数据库级保留策略，并设置表级别的保留策略。

```kusto
.delete table MyTable1 policy retention   // optional, only if the table previously had its policy set
.delete table MyTable2 policy retention   // optional, only if the table previously had its policy set
.alter-merge database MyDatabase policy retention softdelete = 7d recoverability = disabled
.alter-merge table MySpecialTable policy retention softdelete = 14d recoverability = enabled
```

* *选项 2* ：为每个表设置一个表级别保留策略，并设置相关的软删除期和可恢复性。

```kusto
.alter-merge table MyTable1 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MyTable2 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MySpecialTable policy retention softdelete = 14d recoverability = enabled
```

### <a name="soft-delete-period-of-seven-days-and-myspecialtable-keeps-its-data-indefinitely"></a>软删除期为七天，并且 `MySpecialTable` 无限期保留其数据

将表 `MyTable1` 和 `MyTable2` 设置为具有七天的软删除期，并且 `MySpecialTable` 无限期地保留其数据。

* *选项 1* ：为 `MySpecialTable` 设置数据库级别的保留策略，并设置一个软删除期为 100 年的表级别保留策略（默认保留策略）。

```kusto
.delete table MyTable1 policy retention   // optional, only if the table previously had its policy set
.delete table MyTable2 policy retention   // optional, only if the table previously had its policy set
.alter-merge database MyDatabase policy retention softdelete = 7d
.alter table MySpecialTable policy retention "{}" // this sets the default retention policy
```

* *选项 2* ：对于表 `MyTable1` 和表 `MyTable2`，设置表级别的保留策略，并验证 `MySpecialTable` 的数据库级别和表级别策略未设置。

```kusto
.delete database MyDatabase policy retention   // optional, only if the database previously had its policy set
.delete table MySpecialTable policy retention   // optional, only if the table previously had its policy set
.alter-merge table MyTable1 policy retention softdelete = 7d
.alter-merge table MyTable2 policy retention softdelete = 7d
```

* *选项 3* ：对于表 `MyTable1` 和表 `MyTable2`，设置表级别的保留策略。 为表 `MySpecialTable` 设置一个软删除期为 100 年的表级别保留策略（默认保留策略）。

```kusto
.alter-merge table MyTable1 policy retention softdelete = 7d
.alter-merge table MyTable2 policy retention softdelete = 7d
.alter table MySpecialTable policy retention "{}"
```
