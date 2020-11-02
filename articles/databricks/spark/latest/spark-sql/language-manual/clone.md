---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 克隆（Azure Databricks 上的 Delta Lake）- Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Delta Lake SQL 语言的 CLONE 语法。
ms.openlocfilehash: e33940018faf41c52e5bb94c1df68e82bfd17fe4
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472823"
---
# <a name="clone-delta-lake-on-azure-databricks"></a>克隆（Azure Databricks 上的 Delta Lake）

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../../release-notes/release-types.md)提供。

> [!NOTE]
>
> 可在 Databricks Runtime 7.2 及更高版本中使用。

将源 Delta 表克隆到特定版本的目标位置。 克隆可以是深层克隆，也可以是浅层克隆，这取决于克隆是否复制了源中的数据。

> [!IMPORTANT]
>
> 就克隆与源之间的依赖性以及其他差异而言，浅层克隆与深层克隆之间存在重要差异。 请参阅[克隆 Delta 表](../../../../delta/delta-utility.md#clone-delta-table)。

```sql
CREATE TABLE [IF NOT EXISTS] [db_name.]target_table
[SHALLOW | DEEP] CLONE [db_name.]source_table [<time_travel_version>]
[LOCATION 'path']
```

```sql
[CREATE OR] REPLACE TABLE [db_name.]target_table
[SHALLOW|DEEP] CLONE [db_name.]source_table [<time_travel_version>]
[LOCATION 'path']
```

其中

```sql
<time_travel_version>  =
  TIMESTAMP AS OF timestamp_expression |
  VERSION AS OF version
```

* 请指定 `CREATE IF NOT EXISTS` 以避免在表 `target_table` 已存在的情况下创建它。 如果目标位置已存在表，则克隆操作将是一个无操作的操作。
* 请指定 `CREATE OR REPLACE` 以在表 `target_table` 已存在的情况下替换克隆操作的目标。 在已使用表名的情况下，这将使用新表更新元存储。
* 指定 `SHALLOW` 或 `DEEP` 会在目标位置创建浅层或深层克隆。 如果 `SHALLOW` 和 `DEEP` 均未指定，则默认情况下会创建深层克隆。
* 指定 `LOCATION` 会在目标位置创建一个外部表，并将所提供的位置作为要存储数据的路径。 如果所提供的目标是路径而不是表名，则此操作会失败。

## <a name="examples"></a>示例

可以使用 `CLONE` 执行复杂的操作，例如数据迁移、数据存档、机器学习流复现、短期试验、数据共享等。如需一些示例，请参阅[克隆用例](../../../../delta/delta-utility.md#clone-use-cases)。