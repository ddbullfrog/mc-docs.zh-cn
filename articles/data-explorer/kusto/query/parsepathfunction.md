---
title: parse_path() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 parse_path()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 10/29/2020
ms.openlocfilehash: c60a1880b4b3360ab6bd84aff667e2ffe24f213c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106145"
---
# <a name="parse_path"></a>parse_path()

分析文件路径 `string`，并返回包含该路径的以下部分的 [`dynamic`](./scalar-data-types/dynamic.md) 对象：
* Scheme
* RootPath
* DirectoryPath
* DirectoryName
* FileName
* 分机
* AlternateDataStreamName

除了具有两种斜杠类型的简单路径外，此函数还支持包含以下内容的路径：
* 架构。 例如，“file://...”
* 共享路径。 例如，“\\shareddrive\users...”
* 长路径。 例如，“\\?\C:...”
* 备用数据流。 例如，“file1.exe:file2.exe”

## <a name="syntax"></a>语法

`parse_path(`*path*`)`

## <a name="arguments"></a>参数

* *path* ：一个表示文件路径的字符串。

## <a name="returns"></a>返回

包含上面列出的路径组件的 `dynamic` 类型的对象。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(p:string) 
[
    @"C:\temp\file.txt",
    @"temp\file.txt",
    "file://C:/temp/file.txt:some.exe",
    @"\\shared\users\temp\file.txt.gz",
    "/usr/lib/temp/file.txt"
]
| extend path_parts = parse_path(p)

```

|p|path_parts
|---|---
|C:\temp\file.txt|{"Scheme":"","RootPath":"C:","DirectoryPath":"C:\\temp","DirectoryName":"temp","Filename":"file.txt","Extension":"txt","AlternateDataStreamName":""}
|temp\file.txt|{"Scheme":"","RootPath":"","DirectoryPath":"temp","DirectoryName":"temp","Filename":"file.txt","Extension":"txt","AlternateDataStreamName":""}
|file://C:/temp/file.txt:some.exe|{"Scheme":"file","RootPath":"C:","DirectoryPath":"C:/temp","DirectoryName":"temp","Filename":"file.txt","Extension":"txt","AlternateDataStreamName":"some.exe"}
|\\shared\users\temp\file.txt.gz|{"Scheme":"","RootPath":"","DirectoryPath":"\\\\shared\\users\\temp","DirectoryName":"temp","Filename":"file.txt.gz","Extension":"gz","AlternateDataStreamName":""}
|/usr/lib/temp/file.txt|{"Scheme":"","RootPath":"","DirectoryPath":"/usr/lib/temp","DirectoryName":"temp","Filename":"file.txt","Extension":"txt","AlternateDataStreamName":""}
