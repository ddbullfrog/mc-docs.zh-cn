---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 06/18/2020
title: API 参考 - Azure Databricks
description: 了解有关由 Delta Lake 提供的 API 的信息。
ms.openlocfilehash: 0b90bb3b4b0f2671f42440207a5c1cf93fa6164b
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121902"
---
# <a name="api-reference"></a>API 参考

对于 Delta 表上最常见的读写操作，可以使用 Apache Spark 读取器和编写器 API（请参阅[表的批量读取和写入](delta-batch.md)和[表的流式读取和写入](delta-streaming.md)）。 但是，有一些操作是特定于 Delta Lake 的，必须使用 Delta Lake 编程 API。 本文将介绍这些编程 API。

> [!NOTE]
>
> 某些编程 API 仍在不断发展，在 API 文档中用“Evolving”限定符来表示。

Azure Databricks 可确保 Delta Lake 项目与 Databricks Runtime 中的 Delta Lake 之间的二进制兼容性。
[兼容性矩阵](../release-notes/runtime/releases.md#compatibility-matrixes)列出了每个 Databricks Runtime 版本中打包的 Delta Lake API 版本以及指向相应 API 文档的链接。