---
title: Azure 数据资源管理器支持的用于引入的数据格式。
description: 了解 Azure 数据资源管理器支持的用于引入的各种数据和压缩格式。
author: orspod
ms.author: v-tawe
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: conceptual
origin.date: 03/19/2020
ms.date: 09/30/2020
ms.openlocfilehash: 9c67c76a0cfdbf1c79d8e999f616f0822f2db6a8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103614"
---
# <a name="data-formats-supported-by-azure-data-explorer-for-ingestion"></a>Azure 数据资源管理器支持的用于引入的数据格式

数据引入是指在 Azure 数据资源管理器中将数据添加到表中供查询这一过程。 对于 ingest-from-query 以外的所有引入方法，数据必须采用下述受支持格式中的一种。 下表列出并说明了 Azure 数据资源管理器支持的数据引入格式。

|格式   |分机   |说明|
|---------|------------|-----------|
|ApacheAvro|`.avro`    |[Avro 格式](https://avro.apache.org/docs/current/)，支持[逻辑类型](https://avro.apache.org/docs/current/spec.html#Logical+Types)和 `snappy` 压缩编解码器。|
|Avro     |`.avro`     |[Avro 容器文件](https://avro.apache.org/docs/current/)的传统实现。 支持以下代码：`null`、`deflate`（针对 `snappy` - 使用 `apacheavro` 文件格式）。|
|CSV      |`.csv`      |一个采用逗号分隔值 (`,`) 的文本文件。 请参阅 [RFC 4180: _Common Format and MIME Type for Comma-Separated Values (CSV) Files_](https://www.ietf.org/rfc/rfc4180.txt)（RFC 4180：逗号分隔值 (CSV) 文件的常见格式和 MIME 类型）。|
|JSON     |`.json`     |一个文本文件，其中包含使用 `\n` 或 `\r\n` 分隔的 JSON 对象。 请参阅 [JSON Lines (JSONL)](http://jsonlines.org/)。|
|MultiJSON|`.multijson`|一个文本文件，包含一个由属性包（每个包代表一条记录）组成的 JSON 数组，或者包含通过空格、`\n` 或 `\r\n` 分隔的任意数目的属性包。 每个属性包可以分布在多个行上。 此格式优先于 `JSON`，除非数据为非属性包。|
|ORC      |`.orc`      |Orc 文件。|
|Parquet  |`.parquet`  |Parquet 文件。|
|PSV      |`.psv`      |一个采用竖线分隔值 (<code>&#124;</code>) 的文本文件。|
|RAW      |`.raw`      |一个文本文件，其整个内容就是一个字符串值。|
|SCsv     |`.scsv`     |一个采用分号分隔值 (`;`) 的文本文件。|
|SOHsv    |`.sohsv`    |一个采用 SOH 分隔值的文本文件。 （SOH 为 ASCII 代码点 1；此格式由 Hive on HDInsight 使用。）|
|TSV      |`.tsv`      |一个采用制表符分隔值 (`\t`) 的文本文件。|
|TSVE     |`.tsv`      |一个采用制表符分隔值 (`\t`) 的文本文件。 反斜杠字符 (`\`) 用于转义。|
|TXT      |`.txt`      |一个文本文件，使用 `\n` 分隔的行。 空行将被跳过。|
|W3CLOGFILE |`.log`    |由 W3C 标准化的 [Web 日志文件](https://www.w3.org/TR/WD-logfile.html)格式。|

## <a name="supported-data-compression-formats"></a>支持的数据压缩格式

可以使用下面的任何压缩算法来压缩 Blob 和文件：

|压缩|分机|
|-----------|---------|
|GZip       |.gz      |
|Zip        |.zip     |

通过将扩展名追加到 Blob 或文件的名称上来指示压缩。

例如：
* `MyData.csv.zip` 指示格式化为 CSV 且使用 ZIP 进行压缩的 blob 或文件（存档或单个文件）
* `MyData.json.gz` 指示格式化为 JSON 且使用 GZip 进行压缩的 blob 或文件。

不包含格式扩展名而只包含压缩名的 Blob 名或文件名（例如 `MyData.zip`）也受支持。 在这种情况下，必须将文件格式指定为引入属性，因为不能对它进行推断。

> [!NOTE]
> 某些压缩格式会在压缩的流中记录原始文件扩展名。 在确定文件格式时，通常会忽略此扩展名。 如果不能根据（压缩的）Blob 或文件的名称确定文件格式，则必须通过 `format` 引入属性指定它。

## <a name="next-steps"></a>后续步骤

* 详细了解[数据引入](ingest-data-overview.md)
* 详细了解 [Azure 数据资源管理器的数据引入属性](ingestion-properties.md)
