---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 复制到（Azure Databricks 上的 Delta Lake）- Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Delta Lake SQL 语言的 COPY INTO 语法。
ms.openlocfilehash: 28ab6ed99c225fde695ec97b4d29d49266507fdc
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472822"
---
# <a name="copy-into-delta-lake-on-azure-databricks"></a>复制到（Azure Databricks 上的 Delta Lake）

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../../release-notes/release-types.md)提供。

```sql
COPY INTO table_identifier
  FROM [ file_location | (SELECT identifier_list FROM file_location) ]
  FILEFORMAT = data_source
  [FILES = [file_name, ... | PATTERN = 'regex_pattern']
  [FORMAT_OPTIONS ('data_source_reader_option' = 'value', ...)]
  [COPY_OPTIONS 'force' = ('false'|'true')]
```

将文件位置中的数据加载到 Delta 表中。 这是一个可重试的幂等操作 - 跳过源位置中已加载的文件。

**`table_identifier`**

要将数据复制到其中的 Delta 表。

**`FROM file_location`**

要从中加载数据的文件位置。 此位置中的文件必须采用 `FILEFORMAT` 中指定的格式。

**`SELECT identifier_list`**

在复制到 Delta 表之前，从源数据中选择指定的列或表达式。

**`FILEFORMAT = data_source`**

要加载的源文件的格式。 `CSV`、`JSON`、`AVRO`、`ORC`、`PARQUET` 之一。

**`FILES`**

要加载的文件名的列表，长度最大为 1000。 无法使用 `PATTERN` 进行指定。

**`PATTERN`**

正则表达式模式，用于标识要从源目录加载的文件。 无法使用 `FILES` 进行指定。

**`FORMAT_OPTIONS`**

要传递给指定格式的 Apache Spark 数据源读取器的选项。

**`COPY_OPTIONS`**

用于控制 `COPY INTO` 命令的操作的选项。 唯一的选项是 `'force'`；如果设置为 `'true'`，则禁用幂等性并加载文件，而不管文件以前是否加载过。

## <a name="examples"></a>示例

```sql
COPY INTO delta.`target_path`
  FROM (SELECT key, index, textData, 'constant_value' FROM 'source_path')
  FILEFORMAT = CSV
  PATTERN = 'folder1/file_[a-g].csv'
  FORMAT_OPTIONS('header' = 'true')
```