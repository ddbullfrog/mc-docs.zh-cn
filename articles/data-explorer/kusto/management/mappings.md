---
title: 数据映射 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的数据映射。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: ohbitton
ms.service: data-explorer
ms.topic: reference
origin.date: 05/19/2020
ms.date: 09/24/2020
ms.openlocfilehash: 2e4ef89109427d4da340dbfcd343f60a7f0e870f
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146602"
---
# <a name="data-mappings"></a>数据映射

数据映射在引入过程中使用，将传入数据映射到 Kusto 表中的列。

Kusto 支持不同类型的映射，包括 `row-oriented`（CSV、JSON 和 AVRO）和 `column-oriented` (Parquet)。

映射列表中的每个元素都由以下三个属性构成：

|属性|说明|
|----|--|
|`column`|Kusto 表中的目标列名称|
|`datatype`| （可选）数据类型，如果 Kusto 表中不存在映射列时，则使用该数据类型创建映射列|
|`Properties`|（可选）属性包，包含特定于每个映射的属性，如下一节中所述。


所有映射都可以是[预先创建](create-ingestion-mapping-command.md)的，也可以是使用 `ingestionMappingReference` 参数从 ingest 命令引用的。

## <a name="csv-mapping"></a>CSV 映射

如果源文件是 CSV（或任何分隔符分隔的格式），并且其架构与当前 Kusto 表架构不匹配，则 CSV 映射会将文件架构映射到 Kusto 表架构。 如果表在 Kusto 中不存在，则将根据此映射创建该表。 如果表中缺少映射中的某些字段，将添加这些字段。 

CSV 映射可以应用于所有分隔符分隔的格式：CSV、TSV、PSV、SCSV 和 SOHsv。

列表中的每个元素都描述特定列的映射，并且可能包含以下属性：

|属性|说明|
|----|--|
|`ordinal`|CSV 中的列顺序号|
|`constantValue`|（可选）要用于列而不是 CSV 内某个值的常数值|

> [!NOTE]
> `Ordinal` 和 `ConstantValue` 互斥。

### <a name="example-of-the-csv-mapping"></a>CSV 映射示例

``` json
[
  { "column" : "rownumber", "Properties":{"Ordinal":"0"}},
  { "column" : "rowguid",   "Properties":{"Ordinal":"1"}},
  { "column" : "xdouble",   "Properties":{"Ordinal":"2"}},
  { "column" : "xbool",     "Properties":{"Ordinal":"3"}},
  { "column" : "xint32",    "Properties":{"Ordinal":"4"}},
  { "column" : "xint64",    "Properties":{"Ordinal":"5"}},
  { "column" : "xdate",     "Properties":{"Ordinal":"6"}},
  { "column" : "xtext",     "Properties":{"Ordinal":"7"}},
  { "column" : "const_val", "Properties":{"ConstValue":"Sample: constant value"}}
]
```

> [!NOTE]
> 当上述映射作为 `.ingest` 控制命令的一部分提供时，它将被序列化为 JSON 字符串。

* 如果上述映射是[预先创建](create-ingestion-mapping-command.md)的，可使用 `.ingest` 控制命令引用它：

```kusto
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="csv", 
        ingestionMappingReference = "Mapping1"
    )
```

* 当上述映射作为 `.ingest` 控制命令的一部分提供时，它将被序列化为 JSON 字符串：

```kusto
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="csv", 
        ingestionMapping = 
        "["
            "{\"column\":\"rownumber\",\"Properties\":{\"Ordinal\":0}},"
            "{\"column\":\"rowguid\",  \"Properties\":{\"Ordinal\":1}}"
        "]" 
    )
```

**注意：** 不推荐使用以下没有 `Properties` 属性包的映射格式。

```kusto
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="csv", 
        ingestionMapping = 
        "["
            "{\"column\":\"rownumber\",\"Ordinal\": 0},"
            "{\"column\":\"rowguid\",  \"Ordinal\": 1}"
        "]" 
    )
```

## <a name="json-mapping"></a>JSON 映射

当源文件采用 JSON 格式时，文件内容将映射到 Kusto 表。 表必须存在于 Kusto 数据库中，除非为映射的所有列指定有效的数据类型。 在 JSON 映射中，被映射的列必须存在于 Kusto 表中，除非为所有不存在的列指定了数据类型。

列表中的每个元素都描述特定列的映射，并且可能包含以下属性： 

|属性|说明|
|----|--|
|`path`|如果开头为 `$`，指向将成为 JSON 文档中列内容的字段的 JSON 路径（指示整个文档的 JSON 路径为 `$`）。 如果值不以 `$` 开头，则使用常数值。|
|`transform`|（可选）应通过[映射转换](#mapping-transformations)应用于内容的转换。|

### <a name="example-of-json-mapping"></a>JSON 映射示例

```json
[
  { "column" : "rownumber",   "Properties":{"Path":"$.rownumber"}}, 
  { "column" : "rowguid",     "Properties":{"Path":"$.rowguid"}}, 
  { "column" : "xdouble",     "Properties":{"Path":"$.xdouble"}}, 
  { "column" : "xbool",       "Properties":{"Path":"$.xbool"}}, 
  { "column" : "xint32",      "Properties":{"Path":"$.xint32"}}, 
  { "column" : "xint64",      "Properties":{"Path":"$.xint64"}}, 
  { "column" : "xdate",       "Properties":{"Path":"$.xdate"}}, 
  { "column" : "xtext",       "Properties":{"Path":"$.xtext"}}, 
  { "column" : "location",    "Properties":{"transform":"SourceLocation"}}, 
  { "column" : "lineNumber",  "Properties":{"transform":"SourceLineNumber"}}, 
  { "column" : "full_record", "Properties":{"Path":"$"}}
]
```

> [!NOTE]
> 当上述映射作为 `.ingest` 控制命令的一部分提供时，它将被序列化为 JSON 字符串。

```kusto
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="json", 
        ingestionMappingReference = "Mapping1"
    )
```

**注意：** 不推荐使用以下没有 `Properties` 属性包的映射格式。

```kusto
.ingest into Table123 (@"source1", @"source2") 
  with 
  (
      format = "json", 
      ingestionMapping = 
      "["
        "{\"column\":\"rownumber\",\"path\":\"$.rownumber\"},"
        "{\"column\":\"rowguid\",  \"path\":\"$.rowguid\"}"
      "]"
  )
```
    
## <a name="avro-mapping"></a>Avro 映射

当源文件采用 Avro 格式时，Avro 文件内容将映射到 Kusto 表。 表必须存在于 Kusto 数据库中，除非为映射的所有列指定有效的数据类型。 在 Avro 映射中，被映射的列必须存在于 Kusto 表中，除非为所有不存在的列指定了数据类型。

列表中的每个元素都描述特定列的映射，并且可能包含以下属性： 

|属性|说明|
|----|--|
|`Field`|Avro 记录中字段的名称。|
|`Path`|此外还可使用 `field`，它允许采用 Avro 记录字段的内部部分（如有必要）。 该值表示从记录的根开始的 JSON 路径。 有关详细信息，请参阅以下备注。 |
|`transform`|（可选）应通过[支持的转换](#mapping-transformations)应用于内容的转换。|

**说明**
>[!NOTE]
> * `field` 和 `path` 不能一起使用，只允许使用一个。 
> * `path` 不能仅指向根 `$`，它必须具有至少一级路径。

以下两种替代方法是等效的：

``` json
[
  {"column": "rownumber", "Properties":{"Path":"$.RowNumber"}}
]
```

``` json
[
  {"column": "rownumber", "Properties":{"Field":"RowNumber"}}
]
```

### <a name="example-of-the-avro-mapping"></a>AVRO 映射示例

``` json
[
  {"column": "rownumber", "Properties":{"Field":"rownumber"}},
  {"column": "rowguid",   "Properties":{"Field":"rowguid"}},
  {"column": "xdouble",   "Properties":{"Field":"xdouble"}},
  {"column": "xboolean",  "Properties":{"Field":"xboolean"}},
  {"column": "xint32",    "Properties":{"Field":"xint32"}},
  {"column": "xint64",    "Properties":{"Field":"xint64"}},
  {"column": "xdate",     "Properties":{"Field":"xdate"}},
  {"column": "xtext",     "Properties":{"Field":"xtext"}}
]
``` 

> [!NOTE]
> 当上述映射作为 `.ingest` 控制命令的一部分提供时，它将被序列化为 JSON 字符串。

```kusto
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="avro", 
        ingestionMappingReference = "Mapping1"
    )
```

**注意：** 不推荐使用以下没有 `Properties` 属性包的映射格式。

```kusto
.ingest into Table123 (@"source1", @"source2") 
  with 
  (
      format = "avro", 
      ingestionMapping = 
      "["
        "{\"column\":\"rownumber\",\"field\":\"rownumber\"},"
        "{\"column\":\"rowguid\",  \"field\":\"rowguid\"}"
      "]"
  )
```

## <a name="parquet-mapping"></a>Parquet 映射

当源文件采用 Parquet 格式时，文件内容将映射到 Kusto 表。 表必须存在于 Kusto 数据库中，除非为映射的所有列指定有效的数据类型。 在 Parquet 映射中，被映射的列必须存在于 Kusto 表中，除非为所有不存在的列指定了数据类型。

列表中的每个元素都描述特定列的映射，并且可能包含以下属性：

|属性|说明|
|----|--|
|`path`|如果开头为 `$`，指向将成为 Parquet 文档中列内容的字段的 JSON 路径（指示整个文档的 JSON 路径为 `$`）。 如果值不以 `$` 开头，则使用常数值。|
|`transform`|（可选）应该应用于内容的[映射转换](#mapping-transformations)。


### <a name="example-of-the-parquet-mapping"></a>Parquet 映射示例

```json
[
  { "column" : "rownumber",   "Properties":{"Path":"$.rownumber"}}, 
  { "column" : "xdouble",     "Properties":{"Path":"$.xdouble"}}, 
  { "column" : "xbool",       "Properties":{"Path":"$.xbool"}}, 
  { "column" : "xint64",      "Properties":{"Path":"$.xint64"}}, 
  { "column" : "xdate",       "Properties":{"Path":"$.xdate"}}, 
  { "column" : "xtext",       "Properties":{"Path":"$.xtext"}}, 
  { "column" : "location",    "Properties":{"transform":"SourceLocation"}}, 
  { "column" : "lineNumber",  "Properties":{"transform":"SourceLineNumber"}}, 
  { "column" : "full_record", "Properties":{"Path":"$"}}
]
```      

> [!NOTE]
> 当上述映射作为 `.ingest` 控制命令的一部分提供时，它将被序列化为 JSON 字符串。

* 如果上述映射是[预先创建](create-ingestion-mapping-command.md)的，可使用 `.ingest` 控制命令引用它：

```kusto
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="parquet", 
        ingestionMappingReference = "Mapping1"
    )
```

* 当上述映射作为 `.ingest` 控制命令的一部分提供时，它将被序列化为 JSON 字符串：

```kusto
.ingest into Table123 (@"source1", @"source2") 
  with 
  (
      format = "parquet", 
      ingestionMapping = 
      "["
        "{\"column\":\"rownumber\",\"Properties\":{\"Path\":\"$.rownumber\"}},"
        "{\"column\":\"rowguid\",  \"Properties\":{\"Path\":\"$.rowguid\"}}"
      "]"
  )
```

## <a name="orc-mapping"></a>Orc 映射

当源文件采用 Orc 格式时，文件内容将映射到 Kusto 表。 表必须存在于 Kusto 数据库中，除非为映射的所有列指定有效的数据类型。 在 Orc 映射中，被映射的列必须存在于 Kusto 表中，除非为所有不存在的列指定了数据类型。

列表中的每个元素都描述特定列的映射，并且可能包含以下属性：

|属性|说明|
|----|--|
|`path`|如果开头为 `$`，指向将成为 Orc 文档中列内容的字段的 JSON 路径（指示整个文档的 JSON 路径为 `$`）。 如果值不以 `$` 开头，则使用常数值。|
|`transform`|（可选）应该应用于内容的[映射转换](#mapping-transformations)。

### <a name="example-of-orc-mapping"></a>Orc 映射示例

```json
[
  { "column" : "rownumber",   "Properties":{"Path":"$.rownumber"}}, 
  { "column" : "xdouble",     "Properties":{"Path":"$.xdouble"}}, 
  { "column" : "xbool",       "Properties":{"Path":"$.xbool"}}, 
  { "column" : "xint64",      "Properties":{"Path":"$.xint64"}}, 
  { "column" : "xdate",       "Properties":{"Path":"$.xdate"}}, 
  { "column" : "xtext",       "Properties":{"Path":"$.xtext"}}, 
  { "column" : "location",    "Properties":{"transform":"SourceLocation"}}, 
  { "column" : "lineNumber",  "Properties":{"transform":"SourceLineNumber"}}, 
  { "column" : "full_record", "Properties":{"Path":"$"}}
]
```      

> [!NOTE]
> 当上述映射作为 `.ingest` 控制命令的一部分提供时，它将被序列化为 JSON 字符串。

```kusto
.ingest into Table123 (@"source1", @"source2") 
  with 
  (
      format = "orc", 
      ingestionMapping = 
      "["
        "{\"column\":\"rownumber\",\"Properties\":{\"Path\":\"$.rownumber\"}},"
        "{\"column\":\"rowguid\",  \"Properties\":{\"Path\":\"$.rowguid\"}}"
      "]"
  )
```

## <a name="mapping-transformations"></a>映射转换

一些数据格式映射（Parquet、JSON 和 Avro）支持简单且有用的引入时间转换。 如果在引入时需要进行更复杂的处理，可使用[更新策略](update-policy.md)，该策略允许使用 KQL 表达式定义轻型处理。

|路径依赖转换|说明|Conditions|
|--|--|--|
|`PropertyBagArrayToDictionary`|将属性的 JSON 数组（例如 {events:[{"n1":"v1"},{"n2":"v2"}]}）转换为字典，并将其序列化为有效的 JSON 文档（例如 {"n1":"v1","n2":"v2"}）。|仅当使用 `path` 时才能应用|
|`SourceLocation`|提供数据的存储项目的名称，类型 string（例如 blob 的“BaseUri”字段）。|
|`SourceLineNumber`|相对于该存储项目的偏移量，类型 long（从“1”开始，按每条新记录递增）。|
|`DateTimeFromUnixSeconds`|将表示 unix 时间的数字（从 1970-01-01 开始的秒数）转换为 UTC 日期时间字符串|
|`DateTimeFromUnixMilliseconds`|将表示 unix 时间的数字（从 1970-01-01 开始的毫秒数）转换为 UTC 日期时间字符串|
|`DateTimeFromUnixMicroseconds`|将表示 unix 时间的数字（从 1970-01-01 开始的微秒数）转换为 UTC 日期时间字符串|
|`DateTimeFromUnixNanoseconds`|将表示 unix 时间的数字（从 1970-01-01 开始的纳秒数）转换为 UTC 日期时间字符串|
