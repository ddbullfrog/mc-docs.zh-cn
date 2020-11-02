---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 创建函数 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 CREATE FUNCTION 语法。
ms.openlocfilehash: e4fcaafcf770ebb69845bbfd7a75cafd62d4f021
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473081"
---
# <a name="create-function"></a>创建函数

```sql
CREATE [TEMPORARY] FUNCTION [db_name.]function_name AS class_name
  [USING resource, ...]

resource:
  : [JAR|FILE|ARCHIVE] file_uri
```

创建一个函数。 该函数的指定类必须扩展 `org.apache.hadoop.hive.ql.exec` 中的 UDF 或 UDAF，或者扩展 `org.apache.hadoop.hive.ql.udf.generic` 中的 `AbstractGenericUDAFResolver`、`GenericUDF` 或 `GenericUDTF` 之一。 如果数据库中已存在同名的函数，则会引发异常。

> [!NOTE]
>
> 仅当启用了 Hive 支持时，才支持此命令。

**`TEMPORARY`**

创建的函数仅在此会话中可用，并且不会持久保存到基础元存储（如果有）。 不能为临时函数指定数据库名称。

**`USING resource`**

为了支持此函数而必须加载的资源。 JAR、文件或存档 URI 的列表。