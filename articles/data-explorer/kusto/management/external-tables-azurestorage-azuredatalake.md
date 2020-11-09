---
title: 在 Azure 存储或 Azure Data Lake 中创建和更改外部表 - Azure 数据资源管理器
description: 本文介绍了如何在 Azure 存储或 Azure Data Lake 中创建和更改外部表
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 03/24/2020
ms.date: 10/29/2020
ms.openlocfilehash: 5b8fb7b18aade48513a60a17124a196bb12bb890
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103552"
---
# <a name="create-and-alter-external-tables-in-azure-storage-or-azure-data-lake"></a>在 Azure 存储或 Azure Data Lake 中创建和更改外部表

以下命令介绍了如何创建位于 Azure Blob 存储、Azure Data Lake Store Gen1 或 Azure Data Lake Store Gen2 中的外部表。 

有关外部 Azure 存储表功能的简介，请参阅[使用 Azure 数据资源管理器查询 Azure Data Lake 中的数据](../../data-lake-query-data.md)。

## <a name="create-or-alter-external-table"></a>.create 或 .alter external table

**语法**

(`.create` | `.alter` | `.create-or-alter`) `external` `table` *[TableName](#table-name)* `(` *[Schema](#schema)* `)`  
`kind` `=` (`blob` | `adl`)  
[`partition` `by` `(` *[Partitions](#partitions)* `)` [`pathformat` `=` `(` *[PathFormat](#path-format)* `)`]]  
`dataformat` `=` *[Format](#format)*  
`(` *[StorageConnectionString](#connection-string)* [`,` ...] `)`   
[`with` `(`*[PropertyName](#properties)* `=` *[Value](#properties)* `,` ... `)`]  

在执行命令的数据库中创建或更改新的外部表。

> [!NOTE]
> * 如果该表存在，则 `.create` 命令将失败并显示错误。 使用 `.create-or-alter` 或 `.alter` 来修改现有表。
> * 不支持更改外部 blob 表的架构、格式或分区定义。 
> * 此操作需要[数据库用户权限](../management/access-control/role-based-authorization.md)（对于 `.create`）和[表管理员权限](../management/access-control/role-based-authorization.md)（对于 `.alter`）。 

**Parameters**

<a name="table-name"></a>
*TableName*

遵循[实体名称](../query/schema-entities/entity-names.md)规则的外部表名称。
外部表不能与同一数据库中的常规表具有相同的名称。

<a name="schema"></a>
*Schema*

外部数据架构是使用以下格式描述的：

&nbsp;&nbsp;*ColumnName* `:` *ColumnType* [`,` *ColumnName* `:` *ColumnType* ...]

其中，ColumnName 遵循[实体命名](../query/schema-entities/entity-names.md)规则，ColumnType 是[受支持的数据类型](../query/scalar-data-types/index.md)之一。

> [!TIP]
> 如果外部数据架构未知，请使用 [infer\_storage\_schema](../query/inferstorageschemaplugin.md) 插件，该插件有助于根据外部文件内容推断架构。

<a name="partitions"></a>
*Partitions*

对外部表进行分区时所依据的列的逗号分隔列表。 分区列可以存在于数据文件本身或文件路径的 sa 部分中（有关详细信息，请参阅[虚拟列](#virtual-columns)）。

分区列表是分区列的任意组合，使用以下形式之一指定：

* 表示[虚拟列](#virtual-columns)的分区。

  *PartitionName* `:` (`datetime` | `string`)

* 基于字符串列值的分区。

  *PartitionName* `:` `string` `=` *ColumnName*

* 基于字符串列值[哈希](../query/hashfunction.md)且模数为 Number 的分区。

  *PartitionName* `:` `long` `=` `hash` `(` *ColumnName* `,` *Number* `)`

* 基于日期/时间列的截断值的分区。 请参阅有关 [startofyear](../query/startofyearfunction.md)、[startofmonth](../query/startofmonthfunction.md)、[startofweek](../query/startofweekfunction.md)、[startofday](../query/startofdayfunction.md) 或 [bin](../query/binfunction.md) 函数的文档。

  *PartitionName* `:` `datetime` `=` (`startofyear` \| `startofmonth` \| `startofweek` \| `startofday`) `(` *ColumnName* `)`  
  *PartitionName* `:` `datetime` `=` `bin` `(` *ColumnName* `,` *TimeSpan* `)`

若要检查分区定义的正确性，请在创建外部表时使用属性 `sampleUris`。

<a name="path-format"></a>
*PathFormat*

除分区之外还可以指定的外部数据 URI 文件路径格式。 路径格式是分区元素和文本分隔符的序列：

&nbsp;&nbsp;[ *StringSeparator* ] *Partition* [ *StringSeparator* ] [ *Partition* [ *StringSeparator* ] ...]  

其中，Partition 指的是在 `partition` `by` 子句中声明的分区，StringSeparator 是括在引号中的任何文本。 必须使用 StringSeparator 分隔连续的分区元素。

可以使用呈现为字符串并以相应的文本分隔符分隔的分区元素来构造原始文件路径前缀。 若要指定用于呈现日期/时间分区值的格式，可以使用以下宏：

&nbsp;&nbsp;`datetime_pattern` `(` *DateTimeFormat* `,` *PartitionName* `)`  

其中，DateTimeFormat 遵循 .NET 格式规范，并可扩展为允许将格式说明符括在大括号中。 例如，以下两种格式是等效的：

&nbsp;&nbsp;`'year='yyyy'/month='MM` 和 `year={yyyy}/month={MM}`

默认情况下，使用以下格式呈现日期/时间值：

| 分区函数    | 默认格式 |
|-----------------------|----------------|
| `startofyear`         | `yyyy`         |
| `startofmonth`        | `yyyy/MM`      |
| `startofweek`         | `yyyy/MM/dd`   |
| `startofday`          | `yyyy/MM/dd`   |
| `bin(`*Column*`, 1d)` | `yyyy/MM/dd`   |
| `bin(`*Column*`, 1h)` | `yyyy/MM/dd/HH` |
| `bin(`*Column*`, 1m)` | `yyyy/MM/dd/HH/mm` |

如果在外部表定义中省略了 PathFormat，则会假定使用 `/` 分隔符分隔所有分区，分区的顺序与定义它们的顺序完全相同。 分区是使用其默认字符串表示形式呈现的。

若要检查路径格式定义的正确性，请在创建外部表时使用属性 `sampleUris`。

<a name="format"></a>
*Format*

数据格式，可以是任意[引入格式](../../ingestion-supported-formats.md)。

> [!NOTE]
> 将外部表用于[导出方案](data-export/export-data-to-an-external-table.md)仅限于以下格式：`CSV`、`TSV`、`JSON` 和 `Parquet`。

<a name="connection-string"></a>
*StorageConnectionString*

指向 Azure Blob 存储 Blob 容器或 Azure Data Lake Store 文件系统（虚拟目录或文件夹）的一个或多个路径，包括凭据。
有关详细信息，请参阅[存储连接字符串](../api/connection-strings/storage.md)。

> [!TIP]
> 请提供多个存储帐户，以避免将大量数据[导出](data-export/export-data-to-an-external-table.md)到外部表时出现存储限制。 导出会将写入分布到提供的所有帐户之间。 

<a name="properties"></a>
*可选属性*

| 属性         | 类型     | 说明       |
|------------------|----------|-------------------------------------------------------------------------------------|
| `folder`         | `string` | 表的文件夹                                                                     |
| `docString`      | `string` | 用来记录表的字符串                                                       |
| `compressed`     | `bool`   | 如果设置了此项，则表示文件是否压缩为 `.gz` 文件（仅用在[导出方案](data-export/export-data-to-an-external-table.md)中） |
| `includeHeaders` | `string` | 对于带分隔符的文本格式（CSV、TSV、...），指示文件是否包含标头。 可能的值包括：`All`（所有文件都包含标头）、`FirstFile`（文件夹中的第一个文件包含标头）、`None`（无文件包含标头）。 |
| `namePrefix`     | `string` | 如果设置了此项，则表示文件的前缀。 在写入操作中，所有文件都将用此前缀来写入。 在读取操作中，将只读取具有此前缀的文件。 |
| `fileExtension`  | `string` | 如果设置了此项，则表示文件的文件扩展名。 写入时，文件名将以此后缀结尾。 读取时，将只读取具有此文件扩展名的文件。           |
| `encoding`       | `string` | 表示文本编码方式：`UTF8NoBOM`（默认值）或 `UTF8BOM`。             |
| `sampleUris`     | `bool`   | 如果设置了，则命令结果会提供外部表定义所需的外部数据文件 URI 的几个示例（示例在第二个结果表中返回）。 此选项有助于验证是否正确定义了 [Partitions](#partitions) 和 [PathFormat](#path-format) 参数。 |
| `validateNotEmpty` | `bool`   | 如果设置了，则验证连接字符串是否有内容。 如果指定的 URI 位置不存在，或者没有足够的访问它的权限，则该命令会失败。 |

> [!TIP]
> 若要详细了解 `namePrefix` 和 `fileExtension` 属性在查询期间在数据文件筛选中所起的作用，请参阅[文件筛选逻辑](#file-filtering)部分。
 
<a name="examples"></a>
**示例** 

一个未分区的外部表。 数据文件应直接放置在所定义的容器下：

```kusto
.create external table ExternalTable (x:long, s:string)  
kind=blob 
dataformat=csv 
( 
   h@'https://storageaccount.blob.core.chinacloudapi.cn/container1;secretKey'
) 
```

一个按日期分区的外部表。 数据文件应放置在默认日期/时间格式为 `yyyy/MM/dd` 的目录中：

```kusto
.create external table ExternalTable (Timestamp:datetime, x:long, s:string) 
kind=adl
partition by (Date:datetime = bin(Timestamp, 1d)) 
dataformat=csv 
( 
   h@'abfss://filesystem@storageaccount.dfs.core.chinacloudapi.cn/path;secretKey'
)
```

一个按月份分区的外部表，目录格式为 `year=yyyy/month=MM`：

```kusto
.create external table ExternalTable (Timestamp:datetime, x:long, s:string) 
kind=blob 
partition by (Month:datetime = startofmonth(Timestamp)) 
pathformat = (datetime_pattern("'year='yyyy'/month='MM", Month)) 
dataformat=csv 
( 
   h@'https://storageaccount.blob.core.chinacloudapi.cn/container1;secretKey'
) 
```

一个先按客户名称分区然后按日期分区的外部表。 例如，预期的目录结构是 `customer_name=Softworks/2019/02/01`：

```kusto
.create external table ExternalTable (Timestamp:datetime, CustomerName:string) 
kind=blob 
partition by (CustomerNamePart:string = CustomerName, Date:datetime = startofday(Timestamp)) 
pathformat = ("customer_name=" CustomerNamePart "/" Date)
dataformat=csv 
(  
   h@'https://storageaccount.blob.core.chinacloudapi.cn/container1;secretKey'
)
```

一个先按客户名称哈希（模数为 10）分区然后按日期分区的外部表。 例如，预期的目录结构是 `customer_id=5/dt=20190201`。 数据文件名以 `.txt` 扩展名结尾：

```kusto
.create external table ExternalTable (Timestamp:datetime, CustomerName:string) 
kind=blob 
partition by (CustomerId:long = hash(CustomerName, 10), Date:datetime = startofday(Timestamp)) 
pathformat = ("customer_id=" CustomerId "/dt=" datetime_pattern("yyyyMMdd", Date)) 
dataformat=csv 
( 
   h@'https://storageaccount.blob.core.chinacloudapi.cn/container1;secretKey'
)
with (fileExtension = ".txt")
```

**示例输出**

|TableName|TableType|文件夹|DocString|属性|ConnectionStrings|分区|PathFormat|
|---------|---------|------|---------|----------|-----------------|----------|----------|
|ExternalTable|Blob|ExternalTables|Docs|{"Format":"Csv","Compressed":false,"CompressionType":null,"FileExtension":null,"IncludeHeaders":"None","Encoding":null,"NamePrefix":null}|["https://storageaccount.blob.core.chinacloudapi.cn/container1;\*\*\*\*\*\*\*"]|[{"Mod":10,"Name":"CustomerId","ColumnName":"CustomerName","Ordinal":0},{"Function":"StartOfDay","Name":"Date","ColumnName":"Timestamp","Ordinal":1}]|"customer\_id=" CustomerId "/dt=" datetime\_pattern("yyyyMMdd",Date)|

<a name="virtual-columns"></a>
**虚拟列**

从 Spark 导出数据时，不会将分区列（在数据帧编写器的 `partitionBy` 方法中指定）写入到数据文件中。 此过程可避免数据重复，因为数据已存在于“文件夹“名称中。 例如 `column1=<value>/column2=<value>/`，Spark 在读取时可以识别它。

外部表支持用于指定虚拟列的以下语法：

```kusto
.create external table ExternalTable (EventName:string, Revenue:double)  
kind=blob  
partition by (CustomerName:string, Date:datetime)  
pathformat = ("customer=" CustomerName "/date=" datetime_pattern("yyyyMMdd", Date))  
dataformat=parquet
( 
   h@'https://storageaccount.blob.core.chinacloudapi.cn/container1;secretKey'
)
```

<a name="file-filtering"></a>
**文件筛选逻辑**

查询外部表时，查询引擎通过筛选掉无关的外部存储文件来提高性能。 下面介绍了循环访问文件和确定是否应处理某个文件的过程。

1. 生成一个 URI 模式，用于表示找到文件的位置。 最初，URI 模式等于作为外部表定义的一部分提供的连接字符串。 如果定义了任何分区，则它们将使用 *[PathFormat](#path-format)* 呈现，然后追加到 URI 模式。

2. 对于在创建的 URI 模式下找到的所有文件，请检查：

   * 分区值是否与查询中使用的谓词匹配。
   * 如果定义了这样的属性，则 Blob 名称以 `NamePrefix` 开头。
   * 如果定义了这样的属性，则 Blob 名称以 `FileExtension` 结尾。

满足所有条件时，查询引擎将提取并处理该文件。

> [!NOTE]
> 初始 URI 模式是使用查询谓词值构建的。 这最适用于有限的字符串值集以及闭合时间范围。

## <a name="show-external-table-artifacts"></a>.show external table artifacts

返回查询给定外部表时将处理的所有文件的列表。

> [!NOTE]
> 此操作需要[数据库用户权限](../management/access-control/role-based-authorization.md)。

**语法：** 

`.show` `external` `table` *TableName* `artifacts` [`limit` *MaxResults* ]

其中，MaxResults 是可选参数，可设置此参数来限制结果数。

**输出**

| 输出参数 | 类型   | 描述                       |
|------------------|--------|-----------------------------------|
| URI              | string | 外部存储数据文件的 URI |
| 大小             | long   | 文件长度（以字节为单位）              |
| 分区        | 动态 | 描述分区外部表的文件分区的动态对象 |

> [!TIP]
> 循环访问外部表引用的所有文件可能开销会非常高，具体取决于文件的数量。 如果只想查看某些 URI 示例，请确保使用 `limit` 参数。

**示例：**

```kusto
.show external table T artifacts
```

**输出：**

| Uri                                                                     | 大小 | 分区 |
|-------------------------------------------------------------------------| ---- | --------- |
| `https://storageaccount.blob.core.chinacloudapi.cn/container1/folder/file.csv` | 10743 | `{}`   |


对于已分区的表，`Partition` 列将包含提取的分区值：

**输出：**

| Uri                                                                     | 大小 | 分区 |
|-------------------------------------------------------------------------| ---- | --------- |
| `https://storageaccount.blob.core.chinacloudapi.cn/container1/customer=john.doe/dt=20200101/file.csv` | 10743 | `{"Customer": "john.doe", "Date": "2020-01-01T00:00:00.0000000Z"}` |


## <a name="create-external-table-mapping"></a>.create external table mapping

`.create` `external` `table` *ExternalTableName* `json` `mapping` *MappingName* *MappingInJsonFormat*

创建新映射。 有关详细信息，请参阅[数据映射](./mappings.md#json-mapping)。

**示例** 
 
```kusto
.create external table MyExternalTable json mapping "Mapping1" '[{"Column": "rownumber", "Properties": {"Path": "$.rownumber"}}, {"Column": "rowguid", "Properties": {"Path": "$.rowguid"}}]'
```

**示例输出**

| 名称     | 种类 | 映射                                                           |
|----------|------|-------------------------------------------------------------------|
| mapping1 | JSON | [{"ColumnName":"rownumber","Properties":{"Path":"$.rownumber"}},{"ColumnName":"rowguid","Properties":{"Path":"$.rowguid"}}] |

## <a name="alter-external-table-mapping"></a>.alter external table mapping

`.alter` `external` `table` *ExternalTableName* `json` `mapping` *MappingName* *MappingInJsonFormat*

更改现有映射。 
 
**示例** 
 
```kusto
.alter external table MyExternalTable json mapping "Mapping1" '[{"Column": "rownumber", "Properties": {"Path": "$.rownumber"}}, {"Column": "rowguid", "Properties": {"Path": "$.rowguid"}}]'
```

**示例输出**

| 名称     | 种类 | 映射                                                                |
|----------|------|------------------------------------------------------------------------|
| mapping1 | JSON | [{"ColumnName":"rownumber","Properties":{"Path":"$.rownumber"}},{"ColumnName":"rowguid","Properties":{"Path":"$.rowguid"}}] |

## <a name="show-external-table-mappings"></a>.show external table mappings

`.show` `external` `table` *ExternalTableName* `json` `mapping` *MappingName* 

`.show` `external` `table` *ExternalTableName* `json` `mappings`

显示映射（所有映射或按名称指定的某个映射）。
 
**示例** 
 
```kusto
.show external table MyExternalTable json mapping "Mapping1" 

.show external table MyExternalTable json mappings 
```

**示例输出**

| 名称     | 种类 | 映射                                                                         |
|----------|------|---------------------------------------------------------------------------------|
| mapping1 | JSON | [{"ColumnName":"rownumber","Properties":{"Path":"$.rownumber"}},{"ColumnName":"rowguid","Properties":{"Path":"$.rowguid"}}] |

## <a name="drop-external-table-mapping"></a>.drop external table mapping

`.drop` `external` `table` *ExternalTableName* `json` `mapping` *MappingName* 

从数据库中删除映射。
 
**示例** 
 
```kusto
.drop external table MyExternalTable json mapping "Mapping1" 
```
## <a name="next-steps"></a>后续步骤

* [查询外部表](../../data-lake-query-data.md)。
* [将数据导出到外部表](data-export/export-data-to-an-external-table.md)。
* [连续数据导出到外部表](data-export/continuous-data-export.md)。
