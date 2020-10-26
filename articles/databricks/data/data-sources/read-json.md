---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: JSON 文件 - Azure Databricks
description: 了解如何使用 Azure Databricks 读取 JSON 文件中的数据。
ms.openlocfilehash: 6b047c1ebeb343941f706f1fdbbd28df4131dae2
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121918"
---
# <a name="json-files"></a><a id="json"> </a><a id="json-files"> </a>JSON 文件

可在单行或多行模式下读取 JSON 文件。 在单行模式下，可将一个文件拆分成多个部分进行并行读取。

## <a name="single-line-mode"></a>单行模式

在此示例中，每行有一个 JSON 对象：

```json
{"string":"string1","int":1,"array":[1,2,3],"dict": {"key": "value1"}}
{"string":"string2","int":2,"array":[2,4,6],"dict": {"key": "value2"}}
{"string":"string3","int":3,"array":[3,6,9],"dict": {"key": "value3", "extra_key": "extra_value3"}}
```

若要读取 JSON 数据，应使用类似于以下代码示例的内容：

```scala
val df = spark.read.json("example.json")
```

Spark 会自动推断架构。

```scala
df.printSchema
```

```
root
 |-- array: array (nullable = true)
 |    |-- element: long (containsNull = true)
 |-- dict: struct (nullable = true)
 |    |-- extra_key: string (nullable = true)
 |    |-- key: string (nullable = true)
 |-- int: long (nullable = true)
 |-- string: string (nullable = true)
```

## <a name="multi-line-mode"></a>多行模式

如果某个 JSON 对象占用多行，则必须启用 Spark 的多行模式来加载该文件。 文件将作为一个整体实体加载且无法拆分。

```json
[
    {"string":"string1","int":1,"array":[1,2,3],"dict": {"key": "value1"}},
    {"string":"string2","int":2,"array":[2,4,6],"dict": {"key": "value2"}},
    {
        "string": "string3",
        "int": 3,
        "array": [
            3,
            6,
            9
        ],
        "dict": {
            "key": "value3",
            "extra_key": "extra_value3"
        }
    }
]
```

若要读取这样的 JSON 数据，应启用多行模式：

```scala
val mdf = spark.read.option("multiline", "true").json("multi.json")
mdf.show(false)
```

```
+---------+---------------------+---+-------+
|array    |dict                 |int|string |
+---------+---------------------+---+-------+
|[1, 2, 3]|[null,value1]        |1  |string1|
|[2, 4, 6]|[null,value2]        |2  |string2|
|[3, 6, 9]|[extra_value3,value3]|3  |string3|
+---------+---------------------+---+-------+
```

## <a name="charset-auto-detection"></a><a id="charset-auto-detect"> </a><a id="charset-auto-detection"> </a>字符集自动检测

默认情况下，Spark 会自动检测输入文件的字符集，但你始终可通过此选项显式指定字符集：

```python
spark.read.option("charset", "UTF-16BE").json("fileInUTF16.json")
```

下面是一些受支持的字符集：UTF-8、UTF-16BE、UTF-16LE、UTF-16、UTF-32BE、UTF-32LE、UTF-32。 若要查看 Oracle Java SE 支持的字符集的完整列表，请参阅[受支持的编码](https://docs.oracle.com/javase/8/docs/technotes/guides/intl/encoding.doc.html)。

### <a name="read-json-files-notebook"></a>读取 JSON 文件笔记本

[获取笔记本](../../_static/notebooks/read-json-files.html)