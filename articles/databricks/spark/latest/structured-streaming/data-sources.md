---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/10/2020
title: 流式处理数据源和接收器 - Azure Databricks
description: 了解 Azure Databricks 中的结构化流应用程序所支持的流式处理数据源和接收器。
ms.openlocfilehash: d8d46afef81ac9d014540b32b0be2f513cc5c881
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472982"
---
# <a name="streaming-data-sources-and-sinks"></a>流式处理数据源和接收器

结构化流对大量流式处理数据源和接收器（例如文件和 Kafka）提供内置支持，并具有一些程序设计界面，使你能够指定任意数据写入。 以下文章中探讨了这些内容。

* [Apache Kafka](kafka.md)
* [使用自动加载程序从 Azure Blob 存储或 Azure Data Lake Storage Gen2 加载文件](auto-loader.md)
* [Azure 事件中心](streaming-event-hubs.md)
* [Delta Lake 表](delta.md)
* [读取和写入流 Avro 数据](avro-dataframe.md)
* [写入到任意数据接收器](foreach.md)
* [将优化的 Azure Blob 存储文件源与 Azure 队列存储配合使用](aqs.md)