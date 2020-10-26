---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/18/2020
title: Zip 文件 - Azure Databricks
description: 了解如何使用 Azure Databricks 读取 Zip 压缩文件中的数据。
ms.openlocfilehash: 974c81d76e566b0b48f6f691f91431dd28f58681
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121932"
---
# <a name="zip-files"></a><a id="zip"> </a><a id="zip-files"> </a>Zip 文件

Hadoop 不支持作为压缩解压缩程序的 zip 文件。 尽管可将 GZip、BZip2 和其他受支持的压缩格式的文本文件配置为在 Apache Spark 中自动解压缩（只要文件的文件扩展名正确），但是必须执行额外的步骤来读取 zip 文件。

以下笔记本演示如何读取 zip 文件。 将 zip 文件下载到临时目录后，可调用 Azure Databricks `%sh zip` [魔术命令](../../notebooks/notebooks-use.md#language-magic)来解压缩文件。 对于笔记本中使用的示例文件，`tail` 步骤会从解压缩的文件中删除注释行。

使用 `%sh` 对文件进行操作时，结果将存储在 `/databricks/driver` 目录中。 使用 Spark API 加载文件之前，请使用 [Databricks 实用程序](../../dev-tools/databricks-utils.md)将文件移动至 DBFS。

## <a name="zip-files-python-notebook"></a>Zip 文件 Python 笔记本

[获取笔记本](../../_static/notebooks/zip-files-python.html)

## <a name="zip-files-scala-notebook"></a>Zip 文件 Scala 笔记本

[获取笔记本](../../_static/notebooks/zip-files-scala.html)