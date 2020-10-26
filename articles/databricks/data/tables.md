---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/24/2020
title: 数据库和表 - Azure Databricks
description: 了解如何在 Azure Databricks 中查看、创建和管理表和数据库。
ms.openlocfilehash: bf1eb51b50fc94b875856477f9a9bb8c8636f85b
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121950"
---
# <a name="databases-and-tables"></a><a id="databases-and-tables"> </a><a id="tables"> </a>数据库和表

Azure Databricks 数据库是表的集合。 Azure Databricks 表是结构化数据的集合。 可在 Azure Databricks 表上缓存、筛选和执行 Apache Spark [数据帧](../spark/latest/dataframes-datasets/index.md)支持的任何操作。 可使用 [Spark API](../spark/latest/dataframes-datasets/index.md) 和 [Spark SQL](../spark/latest/spark-sql/index.md#spark-sql-lang-manual) 查询表。

有两种类型的表：全局和本地。 全局表可在所有群集中使用。 Azure Databricks 将全局表注册到 Azure Databricks [Hive](https://hive.apache.org/) 元存储或[外部 Hive 元存储](metastores/index.md#metastores)。 有关 Hive 支持的详细信息，请参阅 [Apache Hive 兼容性](../spark/latest/spark-sql/compatibility/hive.md#hive-compatibility)。 本地表不能从其他群集访问，且未在 Hive 元存储中注册。 这也称为临时视图。

可使用[创建表 UI](#create-table-ui) 或[以编程方式](#create-table-programmatically)创建表。 可从 [DBFS](databricks-file-system.md) 中的文件或存储在任何受支持[数据源](data-sources/index.md#data-sources)中的数据填充表。

## <a name="requirements"></a>要求

若要查看和创建数据库和表，必须有一个正在运行的群集。

## <a name="view-databases-and-tables"></a>查看数据库和表

单击边栏中的![数据图标](../_static/images/tables/data-icon.png)。  Azure Databricks 选择一个你有权访问的正在运行的群集。 “数据库”文件夹显示已选定 `default` 数据库的数据库的列表。 “表”文件夹显示 `default` 数据库中的表列表。

> [!div class="mx-imgBorder"]
> ![创建表列表](../_static/images/tables/default-database.png)

可从“数据库”菜单更改群集，[创建表 UI](#create-table-ui) 或[查看表 UI](#view-table-ui)。 例如，在“数据库”菜单中：

1. 单击 ![“数据库”文件夹顶部的](../_static/images/down-caret.png) 向下脱字号。
2. 选择一个群集。

   > [!div class="mx-imgBorder"]
   > ![选择群集](../_static/images/tables/cluster-select.png)

## <a name="create-a-database"></a>创建数据库

在 SQL 中创建数据库：

```sql
CREATE DATABASE <database-name> ...
```

有关更多选项，请参阅[创建数据库](../spark/latest/spark-sql/language-manual/create-database.md)。

## <a name="create-a-table"></a>创建表

可使用 UI 或以编程方式创建表。

### <a name="create-a-table-using-the-ui"></a><a id="create-a-table-using-the-ui"> </a><a id="create-table-ui"> </a>使用 UI 创建表

> [!NOTE]
>
> 使用 UI 创建表时，无法
>
> * 在使用[高并发群集](../clusters/configure.md#high-concurrency)时上传文件。 相反，请使用 [Databricks 文件系统 (DBFS)](databricks-file-system.md) 将数据加载到 Azure Databricks 中。
> * 稍后更新该表。 相反，[以编程方式](#create-table-programmatically)创建表。

> [!NOTE]
>
> 管理员用户可禁用此功能。 请查看[管理数据上传](../administration-guide/workspace/dbfs-ui-upload.md)。

使用 UI 创建表时，将创建一个全局表。

1. 单击 ![边栏中的](../_static/images/tables/data-icon.png) 数据图标。 显示“数据库”和“表”文件夹。
2. 在“数据库”文件夹中，选择一个数据库。
3. 在“表”文件夹的上方，单击“添加数据”。

   ![“添加表”图标](../_static/images/tables/add-table-icon.png)

4. 选择一个数据源，然后按照步骤配置该表。

   > [!IMPORTANT]
   >
   > 表名只能包含小写字母数字字符和下划线，且必须以小写字母或下划线开头。

   > [!div class="mx-imgBorder"]
   > ![配置表](../_static/images/tables/import-table-azure.png)

   **上传文件**

   1. 将文件拖动到“文件”放置区域，或者单击放置区域以浏览并选择文件。 上传后，将为每个文件显示一个路径。 路径将类似于 `/FileStore/tables/<filename>-<random-number>.<file-type>`，可在笔记本中使用此路径来读取数据。

      > [!div class="mx-imgBorder"]
      > ![文件放置区域](../_static/images/data-import/data-import-files.png)

   1. 单击“使用 UI 创建表”。
   1. 在群集下拉列表中，选择一个群集。

   **DBFS**

   1. 选择一个文件。
   1. 单击“使用 UI 创建表”。
   1. 在群集下拉列表中，选择一个群集。
5. 单击“预览表”以查看表。
6. 在“表名称”字段中，可选择性地替代默认表名称。
7. 在“在数据库中创建”字段中，可选择性地替代所选的 `default` 数据库。
8. 在“文件类型”字段中，可选择性地替代推断得到的文件类型。
9. 如果文件类型为 CSV：
   1. 在“列分隔符”字段中，选择是否要替代推断得到的分隔符。
   1. 指示是否将第一行用作列标题。
   1. 指示是否推断架构。
10. 如果文件类型为 JSON，则指示文件是否为多行文件。
11. 单击“创建表”。

#### <a name="create-a-table-in-a-notebook"></a>在笔记本中创建表

在“创建新表 UI”中，可使用 Azure Databricks 提供的快速入门笔记本连接到任何数据源。

* DBFS：单击“在笔记本中创建表”。
* 其他数据源：在“连接器”下拉列表中，选择一种数据源类型。 然后，单击“在笔记本中创建表”。

### <a name="create-a-table-programmatically"></a><a id="create-a-table-programmatically"> </a><a id="create-table-programmatically"> </a>以编程方式创建表

本部分介绍如何以编程方式创建全局表和本地表。

#### <a name="create-a-global-table"></a>创建全局表

在 SQL 中创建全局表：

```sql
CREATE TABLE <table-name> ...
```

有关更多选项，请参阅[创建表](../spark/latest/spark-sql/language-manual/create-table.md)。

用 Python 或 Scala 从数据帧创建全局表：

```python
dataFrame.write.saveAsTable("<table-name>")
```

#### <a name="create-a-local-table"></a><a id="create-a-local-table"> </a><a id="local-table"> </a>创建本地表

用 Python 或 Scala 从数据帧创建本地表：

```python
dataFrame.createOrReplaceTempView("<table-name>")
```

以下示例根据 [Databricks 文件系统 (DBFS)](databricks-file-system.md) 的文件创建一个名为 `diamonds` 的本地表：

```python
dataFrame = "/databricks-datasets/Rdatasets/data-001/csv/ggplot2/diamonds.csv"
spark.read.format("csv").option("header","true")\
  .option("inferSchema", "true").load(dataFrame)\
  .createOrReplaceTempView("diamonds")
```

## <a name="access-a-table"></a>访问表

可查看表详细信息，并读取、更新和删除表。

### <a name="view-table-details"></a><a id="view-table-details"> </a><a id="view-table-ui"> </a>查看表详细信息

表详细信息视图显示了表架构和示例数据。

1. 单击 ![边栏中的](../_static/images/tables/data-icon.png) 数据图标。
2. 在“数据库”文件夹中单击数据库。
3. 在“表”文件夹中单击表名称。
4. 在“群集”下拉列表中，可另外选择一个要呈现表预览的群集。

   > [!div class="mx-imgBorder"]
   > ![表详细信息](../_static/images/tables/tables-view.png)

   > [!NOTE]
   >
   > 为了显示表预览，Spark SQL 查询在“群集”下拉列表中选择的群集上运行。 如果群集上已有工作负载正在运行，则可能需要较长时间才能加载表预览。

### <a name="query-a-table"></a>查询表

这些示例演示如何查询和显示名为 `diamonds` 的表。

#### <a name="sql"></a>SQL

```sql
SELECT * FROM diamonds
```

#### <a name="python"></a>Python

```python
diamonds = spark.sql("select * from diamonds")
display(diamonds.select("*"))

diamonds = spark.table("diamonds")
display(diamonds.select("*"))
```

#### <a name="r"></a>R

```r
diamonds <- sql(sqlContext, "select * from diamonds")
display(diamonds)

diamonds <- table(sqlContext, "diamonds")
display(diamonds)
```

#### <a name="scala"></a>Scala

```scala
val diamonds = spark.sql("select * from diamonds")
display(diamonds.select("*"))

val diamonds = spark.table("diamonds")
display(diamonds.select("*"))
```

## <a name="update-a-table"></a>更新表

表架构是不可变的。 不过，可通过更改基础文件来更新表数据。

例如，对于从存储目录创建的表，添加或删除该目录中的文件将更改表的内容。

更新表的基础文件后，请使用以下命令刷新表：

```sql
REFRESH TABLE <table-name>
```

这可确保在访问表时，Spark SQL 会读取正确的文件，即使基础文件发生更改也是如此。

## <a name="delete-a-table"></a><a id="delete-a-table"> </a><a id="delete-tables"> </a>删除表

###  <a name="delete-a-table-using-the-ui"></a>使用 UI 删除表

1. 单击 ![边栏中的](../_static/images/tables/data-icon.png) 数据图标。
2. 单击表名称旁边的![菜单下拉列表](../_static/images/menu-dropdown.png)，然后选择“删除”。

### <a name="delete-a-table-programmatically"></a>以编程方式删除表

`DROP TABLE <table-name>`

## <a name="managed-and-unmanaged-tables"></a><a id="managed-and-unmanaged-tables"> </a><a id="managed-unmanaged-tables"> </a>托管表和非托管表

每个 Spark SQL 表都具有存储架构和数据本身的元数据信息。

托管表是一个 Spark SQL 表，由 Spark 管理它的数据和元数据。 对于托管表，Databricks 将元数据和数据存储在帐户中的 DBFS 中。 表由 Spark SQL 进行管理，因此执行 `DROP TABLE example_data` 会删除元数据和数据。

创建托管表的一些常见方法是：

### <a name="sql"></a>SQL

```sql
CREATE TABLE <example-table>(id STRING, value STRING)
```

### <a name="python"></a>Python

```python
dataframe.write.saveAsTable("<example-table>")
```

另一种选择是让 Spark SQL 管理元数据，由你来控制数据位置。 这称为非托管表。
由 Spark SQL 管理相关元数据，因此在你执行 `DROP TABLE <example-table>` 时，Spark 只删除元数据，不删除数据本身。 数据仍存在于你提供的路径中。

可在数据源（例如 Cassandra、JDBC 表等）中创建包含数据的非托管表。 若要详细了解 Databricks 支持的数据源，请参阅[数据源](data-sources/index.md)。 创建非托管表的一些常见方法是：

### <a name="sql"></a>SQL

```sql
CREATE TABLE <example-table>(id STRING, value STRING) USING org.apache.spark.sql.parquet OPTIONS (PATH "<your-storage-path>")
```

### <a name="python"></a>Python

```python
dataframe.write.option('path', "<your-storage-path>").saveAsTable("<example-table>")
```

### <a name="replace-table-contents"></a>替换表内容

可通过两种主要方法来替换表内容：简单方法和推荐方法。

#### <a name="simple-way-to-replace-table-contents"></a>用于替换表内容的简单方法

要替换表内容，最简单的方法是删除表元数据和数据，并再创建一个表。

##### <a name="managed-table"></a>托管表

```sql
DROP TABLE IF EXISTS <example-table>     // deletes the metadata and data
CREATE TABLE <example-table> AS SELECT ...
```

##### <a name="unmanaged-table"></a>非托管表

```python
DROP TABLE IF EXISTS <example-table>         // deletes the metadata
dbutils.fs.rm("<your-storage-path>", true)   // deletes the data
CREATE TABLE <example-table> using org.apache.spark.sql.parquet OPTIONS (path "<your-storage-path>") AS SELECT ...
```

另一种方法是：

1. 使用 SQL DDL 创建表：

   ```sql
   CREATE TABLE <table-name> (id long, date string) USING PARQUET LOCATION "<storage-location>"
   ```

2. 将新数据存储在 `<storage-location>` 中。
3. 运行 `refresh table <table-name>`。

#### <a name="recommended-way-to-replace-table-contents"></a>用于替换表内容的推荐方法

为了避免潜在的一致性问题，替换表内容的最佳方法是覆盖表。

##### <a name="python"></a>Python

```python
dataframe.write.mode("overwrite").saveAsTable("<example-table>") // Managed Overwrite
dataframe.write.mode("overwrite").option("path","<your-storage-path>").saveAsTable("<example-table>")  // Unmanaged Overwrite
```

##### <a name="sql"></a>SQL

使用 `insert overwrite` 关键字。 此方法适用于托管表和非托管表。 例如，对于非托管表：

```sql
CREATE TABLE <example-table>(id STRING, value STRING) USING org.apache.spark.sql.parquet OPTIONS (PATH "<your-storage-path>")
INSERT OVERWRITE TABLE <example-table> SELECT ...
```

##### <a name="scala"></a>Scala

```scala
dataframe.write.mode(SaveMode.Overwrite).saveAsTable("<example-table>")    // Managed Overwrite
dataframe.write.mode(SaveMode.Overwrite).option("path", "<your-storage-path>").saveAsTable("<example-table>")  // Unmanaged Overwrite
```

## <a name="partitioned-tables"></a>分区表

Spark SQL 能够在文件存储级别动态生成分区，以便为表提供分区列。

### <a name="create-a-partitioned-table"></a>创建已分区表

这些示例对写入的数据进行分区。 Spark SQL 会发现分区，并在 Hive 元存储中注册它们。

```scala
// Create managed table as select
dataframe.write.mode(SaveMode.Overwrite).partitionBy("id").saveAsTable("<example-table>")

// Create unmanaged/external table as select
dataframe.write.mode(SaveMode.Overwrite).option("path", "<file-path>").saveAsTable("<example-table>")
```

但是，如果从现有数据创建已分区表，Spark SQL 不会自动发现分区，也不会在 Hive 元存储中注册它们。 在这种情况下，`SELECT * FROM <example-table>` 不会返回结果。 若要注册分区，请运行以下命令来生成分区：`MSCK REPAIR TABLE "<example-table>"`。

```scala
// Save data to external files
dataframe.write.mode(SaveMode.Overwrite).partitionBy("id").parquet("<file-path>")

// Create unmanaged/external table
spark.sql("CREATE TABLE <example-table>(id STRING, value STRING) USING parquet PARTITIONED BY(id) LOCATION "<file-path>"")
spark.sql("MSCK REPAIR TABLE "<example-table>"")
```

### <a name="partition-pruning"></a>删除分区

扫描表时，Spark 会向下推送涉及 `partitionBy` 键的筛选器谓词。 在这种情况下，Spark 避免读取不符合这些谓词的数据。 例如，假设有一个按 `<date>` 分区的表 `<example-data>`。 查询（如 `SELECT max(id) FROM <example-data> WHERE date = '2010-10-10'`）仅读取包含其 `date` 值与查询中指定的元组相匹配的元组的数据文件。

## <a name="table-access-control"></a>表访问控制

通过表访问控制，管理员和用户可向其他用户提供精细化的访问权限。 有关详细信息，请参阅[数据对象特权](../security/access-control/table-acls/object-privileges.md)。