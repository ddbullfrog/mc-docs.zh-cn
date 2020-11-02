---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/04/2020
title: Apache Hive 兼容性 - Azure Databricks
description: 了解与 Apache Hive 兼容的 Azure Databricks 中的 Apache Spark SQL 语言功能。
ms.openlocfilehash: 4a5b524e1a9963a7532e7913c61b32ff884e1450
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473115"
---
# <a name="apache-hive-compatibility"></a><a id="apache-hive-compatibility"> </a><a id="hive-compatibility"> </a>Apache Hive 兼容性

Azure Databricks 中的 Apache Spark SQL 已设计为与 Apache Hive（包括元存储连接性、SerDe 和 UDF）兼容。

## <a name="serdes-and-udfs"></a>SerDe 和 UDF

Hive SerDe 和 UDF 基于 Hive 1.2.1。

## <a name="metastore-connectivity"></a>元存储连接性

请参阅[外部 Apache Hive 元存储](../../../../data/metastores/external-hive-metastore.md)，以了解如何将 Azure Databricks 连接到外部托管的 Hive 元存储。

## <a name="supported-hive-features"></a>支持的 Hive 功能

Spark SQL 支持绝大多数 Hive 功能，如：

* Hive 查询语句，包括：
  * SELECT
  * GROUP BY
  * ORDER BY
  * CLUSTER BY
  * 排序依据
* 所有 Hive 表达式，包括：
  * 关系表达式（`=`、`⇔`、`==`、`<>`、`<`、`>`、`>=`、`<=`，等等）
  * 算术表达式（`+`、`-`、`*`、`/`、`%`，等等）
  * 逻辑表达式（AND、&&、OR、||，等等）
  * 复杂类型构造函数
  * 数学表达式（sign、ln、cos，等等）
  * 字符串表达式（instr、length、printf，等等）
* 用户定义的函数 (UDF)
* 用户定义的聚合函数 (UDAF)
* 用户定义的序列化格式 (SerDe)
* 开窗函数
* 联接
  * JOIN
  * {LEFT|RIGHT|FULL} OUTER JOIN
  * LEFT SEMI JOIN
  * CROSS JOIN
* Unions
* 子查询
  * SELECT col FROM ( SELECT a + b AS col from t1) t2
* 采样
* 说明
* 包括动态分区插入的已分区表
* 查看
* 绝大部分 DDL 语句，包括：
  * CREATE TABLE
  * CREATE TABLE AS SELECT
  * ALTER TABLE
* 大多数 Hive 数据类型，包括：
  * TINYINT
  * SMALLINT
  * INT
  * BIGINT
  * BOOLEAN
  * FLOAT
  * DOUBLE
  * STRING
  * BINARY
  * TIMESTAMP
  * DATE
  * ARRAY<>
  * MAP<>
  * STRUCT<>

## <a name="unsupported-hive-functionality"></a>不受支持的 Hive 功能

以下各部分包含了 Spark SQL 不支持的 Hive 功能的列表。 其中的大多数功能在 Hive 部署中很少使用。

### <a name="major-hive-features"></a>主要 Hive 功能

* 写入到由 Hive 创建的通过 Bucket 进行存储的表
* ACID 细化的更新

### <a name="esoteric-hive-features"></a>复杂的 Hive 功能

* 联合类型
* 唯一联接
* 列统计信息收集：Spark SQL 目前不会借助扫描来收集列统计信息，并且只支持对 Hive 元存储的 sizeInBytes 字段进行填充

### <a name="hive-input-and-output-formats"></a>Hive 输入和输出格式

* 适用于 CLI 的文件格式：对于显示回 CLI 中的结果，Spark SQL 只支持 TextOutputFormat
* Hadoop 存档

### <a name="hive-optimizations"></a>Hive 优化

Spark 中未包括少量的 Hive 优化。 其中一些（如索引）并不太重要，因为 Spark SQL 有内存中计算模型。

* 块级别位图索引和虚拟列（用于生成索引）。
* 自动为 join 和 groupby 确定减速器的数量：在 Spark SQL 中，需要使用 `SET spark.sql.shuffle.partitions=[num_tasks];` 来控制在执行 shuffle 操作后的并行度。
* 倾斜数据标志：Spark SQL 不会遵循 Hive 中的倾斜数据标志。
* join 中的 `STREAMTABLE` 提示：Spark SQL 不会遵循 `STREAMTABLE` 提示。
* 合并查询结果的多个小文件：如果结果输出包含多个小文件，Hive 可以选择性地将这些小文件合并为较少的大文件，以避免溢出 HDFS 元数据。 Spark SQL 不支持此功能。