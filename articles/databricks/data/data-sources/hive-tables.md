---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/18/2020
title: Hive 表 - Azure Databricks
description: 了解如何使用 Azure Databricks 导入存储在云存储中的 Apache Hive 表。
ms.openlocfilehash: 120f0f0d971781293f99868f28455386400be6ef
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121823"
---
# <a name="hive-tables"></a>Hive 表

本文介绍如何使用外部表将 [Hive 表](https://spark.apache.org/docs/latest/sql-data-sources-hive-tables.html)从云存储导入 Azure Databricks。 必须将云存储中的表装载到 [Databricks 文件系统 (DBFS)](../databricks-file-system.md)。

## <a name="step-1-show-the-create-table-statement"></a>步骤 1：显示 `CREATE TABLE` 语句

在 Hive 命令行上发出 `SHOW CREATE TABLE <tablename>` 命令，以查看创建了此表的语句。

```sql
hive> SHOW CREATE TABLE wikicc;
OK
CREATE  TABLE `wikicc`(
  `country` string,
  `count` int)
ROW FORMAT SERDE
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
STORED AS INPUTFORMAT
  'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  '/user/hive/warehouse/wikicc'
TBLPROPERTIES (
  'totalSize'='2335',
  'numRows'='240',
  'rawDataSize'='2095',
  'COLUMN_STATS_ACCURATE'='true',
  'numFiles'='1',
  'transient_lastDdlTime'='1418173653')
```

## <a name="step-2-issue-a-create-external-table-statement"></a>步骤 2：发出 `CREATE EXTERNAL TABLE` 语句

如果返回的语句使用 `CREATE TABLE` 命令，请复制该语句并将 `CREATE TABLE` 替换为 `CREATE EXTERNAL TABLE`。

* `EXTERNAL` 可确保在删除表时 Spark SQL 不会删除你的数据。
* 可以省略 `TBLPROPERTIES` 字段。

```sql
DROP TABLE wikicc
```

```sql
CREATE EXTERNAL TABLE `wikicc`(
  `country` string,
  `count` int)
ROW FORMAT SERDE
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
STORED AS INPUTFORMAT
  'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  '/user/hive/warehouse/wikicc'
```

## <a name="step-3-issue-sql-commands-on-your-data"></a>步骤 3：对数据发出 SQL 命令

```sql
SELECT * FROM wikicc
```