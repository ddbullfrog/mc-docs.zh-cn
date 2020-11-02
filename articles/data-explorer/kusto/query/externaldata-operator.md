---
title: externaldata 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的外部数据运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/24/2020
ms.date: 10/29/2020
ms.openlocfilehash: acfe58b5f269e35e7667b63cccb4a38475f17c85
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105157"
---
# <a name="externaldata-operator"></a>externaldata 运算符

`externaldata` 运算符返回一个表，该表的架构是在查询自身中定义的，并且该表的数据是从外部存储项目（如 Azure Blob 存储中的 Blob 或 Azure Data Lake Storage 中的文件）中读取的。

## <a name="syntax"></a>语法

`externaldata` `(` *ColumnName* `:` *ColumnType* [`,` ...] `)`   
`[` *StorageConnectionString* [`,` ...] `]`   
[`with` `(` PropertyName `=` PropertyValue [`,` ...] `)`] 

## <a name="arguments"></a>参数

* *ColumnName* , *ColumnType* ：这些参数定义表的架构。
  该语法与定义 [.create table](../management/create-table-command.md) 中的表时所使用的语法相同。

* StorageConnectionString：[存储连接字符串](../api/connection-strings/storage.md)，用于描述包含要返回的数据的存储项目。

* *PropertyName* , *PropertyValue* , ...：其他属性（如 [引入属性](../../ingestion-properties.md)下面所列），描述如何解释从存储中检索到的数据。

当前支持的属性包括：

| 属性         | 类型     | 说明       |
|------------------|----------|-------------------|
| `format`         | `string` | 数据格式。 如果未指定，则会尝试从文件扩展名检测数据格式（默认为 `CSV`）。 支持任何[引入数据格式](../../ingestion-supported-formats.md)。 |
| `ignoreFirstRecord` | `bool` | 如果设置为 true，则表示每个文件中的第一条记录均会被忽略。 在查询带有标题的 CSV 文件时，此属性很有用。 |
| `ingestionMapping` | `string` | 一个字符串值，表示如何将数据从源文件映射到运算符结果集中的实际列。 请参阅[数据映射](../management/mappings.md)。 |


> [!NOTE]
> * 此运算符不接受任何管道输入。
> * 标准[查询限制](../concepts/querylimits.md)还适用于外部数据查询。

## <a name="returns"></a>返回

`externaldata` 运算符返回给定架构的数据表，表中的数据是从指定的存储项目中分析的，由存储连接字符串指示。

## <a name="examples"></a>示例

**提取存储在 Azure Blob 存储中的用户 ID 的列表**

下面的示例显示了如何查找表中的所有记录，该表的 `UserID` 列属于一个已知 ID 集，这些 ID 保存在外部存储文件中（每行一个 ID）。 由于未指定数据格式，因此检测到的数据格式是 `TXT`。

```kusto
Users
| where UserID in ((externaldata (UserID:string) [
    @"https://storageaccount.blob.core.chinacloudapi.cn/storagecontainer/users.txt"
      h@"?...SAS..." // Secret token needed to access the blob
    ]))
| ...
```

**查询多个数据文件**

下面的示例查询外部存储中存储的多个数据文件。

```kusto
externaldata(Timestamp:datetime, ProductId:string, ProductDescription:string)
[
  h@"https://mycompanystorage.blob.core.chinacloudapi.cn/archivedproducts/2019/01/01/part-00000-7e967c99-cf2b-4dbb-8c53-ce388389470d.csv.gz?...SAS...",
  h@"https://mycompanystorage.blob.core.chinacloudapi.cn/archivedproducts/2019/01/02/part-00000-ba356fa4-f85f-430a-8b5a-afd64f128ca4.csv.gz?...SAS...",
  h@"https://mycompanystorage.blob.core.chinacloudapi.cn/archivedproducts/2019/01/03/part-00000-acb644dc-2fc6-467c-ab80-d1590b23fc31.csv.gz?...SAS..."
]
with(format="csv")
| summarize count() by ProductId
```

可将上述示例视为快速查询多个数据文件（无需定义[外部表](schema-entities/externaltables.md)）的方法。

> [!NOTE]
> `externaldata` 运算符无法识别数据分区。

**查询分层数据格式**

若要查询分层数据格式（如 `JSON`、 `Parquet`、 `Avro`或 `ORC`），必须在运算符属性中指定 `ingestionMapping`。 在此示例中，有一个 JSON 文件存储在 Azure Blob 存储中，该文件包含以下内容：

```JSON
{
  "timestamp": "2019-01-01 10:00:00.238521",   
  "data": {    
    "tenant": "e1ef54a6-c6f2-4389-836e-d289b37bcfe0",   
    "method": "RefreshTableMetadata"   
  }   
}   
{
  "timestamp": "2019-01-01 10:00:01.845423",   
  "data": {   
    "tenant": "9b49d0d7-b3e6-4467-bb35-fa420a25d324",   
    "method": "GetFileList"   
  }   
}
...
```

若要使用 `externaldata` 运算符查询此文件，必须指定数据映射。 该映射指示如何将 JSON 字段映射到运算符结果集列：

```kusto
externaldata(Timestamp: datetime, TenantId: guid, MethodName: string)
[ 
   h@'https://mycompanystorage.blob.core.chinacloudapi.cn/events/2020/09/01/part-0000046c049c1-86e2-4e74-8583-506bda10cca8.json?...SAS...'
]
with(format='multijson', ingestionMapping='[{"Column":"Timestamp","Properties":{"Path":"$.time"}},{"Column":"TenantId","Properties":{"Path":"$.data.tenant"}},{"Column":"MethodName","Properties":{"Path":"$.data.method"}}]')
```

此处使用 `MultiJSON` 格式，因为单个 JSON 记录跨越多行。

有关映射语法的详细信息，请参阅[数据映射](../management/mappings.md)。
