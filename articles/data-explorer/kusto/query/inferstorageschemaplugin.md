---
title: infer_storage_schema 插件 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 infer_storage_schema 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/24/2020
ms.date: 10/29/2020
ms.openlocfilehash: 643802a23a47b8e9903f874ae5ba4ab21bd6a65c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106338"
---
# <a name="infer_storage_schema-plugin"></a>infer_storage_schema 插件

此插件推断外部数据的架构，并以 CSL 架构字符串的形式返回该架构。 在[创建外部表](../management/external-tables-azurestorage-azuredatalake.md#create-or-alter-external-table)时可以使用该字符串。

```kusto
let options = dynamic({
  'StorageContainers': [
    h@'https://storageaccount.blob.core.chinacloudapi.cn/container1;secretKey'
  ],
  'DataFormat': 'parquet',
  'FileExtension': '.parquet'
});
evaluate infer_storage_schema(options)
```

## <a name="syntax"></a>语法

`evaluate` `infer_storage_schema(` *选项* `)`

## <a name="arguments"></a>参数

单一的“选项”参数是 `dynamic` 类型的常数值，该值保留用于指定请求属性的属性包：

|名称                    |必须|描述|
|------------------------|--------|-----------|
|`StorageContainers`|是|[存储连接字符串](../api/connection-strings/storage.md)的列表，这些字符串表示存储的数据项目的前缀 URI|
|`DataFormat`|是|受支持的[数据格式](../../ingestion-supported-formats.md)之一。|
|`FileExtension`|否|只扫描以此文件扩展名结尾的文件。 该参数不是必需的，但指定该参数可能会加快进程速度（或消除数据读取问题）|
|`FileNamePrefix`|否|只扫描以此前缀开头的文件。 该参数不是必需的，但指定该参数可能会加快进程速度|
|`Mode`|否|架构推理策略，`any`、`last` 和 `all` 之一。 分别从任意（找到的第一个）文件、从上一个写入的文件或者从所有文件来推断数据架构。 默认值为 `last`。|

## <a name="returns"></a>返回

`infer_storage_schema` 插件返回一个结果表，其中包含一个保留了 CSL 架构字符串的行/列。

> [!NOTE]
> * 除了“读取”的权限外，存储容器 URI 密钥还必须具有“列表”的权限 。
> * 架构推理策略“all”是非常“昂贵”的运算，因为它意味着要从所有找到的项目中读取并合并它们的架构。
> * 由于错误的类型推测（或者由于架构合并进程），有些返回的类型可能并不是实际的类型。 因此，在创建外部表之前，应该先仔细查看结果。

## <a name="example"></a>示例

```kusto
let options = dynamic({
  'StorageContainers': [
    h@'https://storageaccount.blob.core.chinacloudapi.cn/MovileEvents/2015;secretKey'
  ],
  'FileExtension': '.parquet',
  'FileNamePrefix': 'part-',
  'DataFormat': 'parquet'
});
evaluate infer_storage_schema(options)
```

*结果*

|CslSchema|
|---|
|app_id:string, user_id:long, event_time:datetime, country:string, city:string, device_type:string, device_vendor:string, ad_network:string, campaign:string, site_id:string, event_type:string, event_name:string, organic:string, days_from_install:int, revenue:real|
