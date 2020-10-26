---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 将数据引入到 Delta Lake - Azure Databricks
description: 了解将数据引入到 Delta Lake 的不同方式。
ms.openlocfilehash: b1d3129a2bdd3295c488065fa58cd6b65be8b018
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121888"
---
# <a name="ingest-data-into-delta-lake"></a>将数据引入到 Delta Lake

Azure Databricks 提供多种方法来帮助你将数据引入到 Delta Lake。

## <a name="partner-integrations"></a>合作伙伴集成

可以通过合作伙伴数据集成将数据加载到合作伙伴产品 UI 的 Azure Databricks 中。 这样就可以将数据从各种源引入 Azure Databricks，其特点是低代码、易于实现且可缩放。 有关详细信息，请参阅[合作伙伴数据集成](../integrations/ingestion/index.md)。

## <a name="copy-into-sql-command"></a>`COPY INTO` SQL 命令

通过 `COPY INTO` SQL 命令，你可以将文件位置中的数据加载到 Delta 表中。 这是一个可重试的幂等操作 - 跳过源位置中已加载的文件。 有关详细信息，请参阅 [Copy Into（Azure Databricks 上的 Delta Lake）](../spark/latest/spark-sql/language-manual/copy-into.md)。

## <a name="auto-loader"></a>自动加载程序

自动加载程序会在新数据文件到达云存储空间时以增量方式高效地对其进行处理，而无需进行任何其他设置。  自动加载程序提供了名为 `cloudFiles` 的新的结构化流式处理源。 给定云文件存储上的输入目录路径后，`cloudFiles` 源将在新文件到达时自动处理这些文件，你也可以选择处理该目录中的现有文件。 有关详细信息，请参阅[使用自动加载程序从 Azure Blob 存储或 Azure Data Lake Storage Gen2 加载文件](../spark/latest/structured-streaming/auto-loader.md)。