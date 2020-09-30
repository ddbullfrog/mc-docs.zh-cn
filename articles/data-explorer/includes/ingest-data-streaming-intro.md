---
title: include 文件
description: include 文件
author: orspod
ms.service: data-explorer
ms.topic: include
origin.date: 07/13/2020
ms.date: 08/18/2020
ms.author: v-tawe
ms.reviewer: alexefro
ms.custom: include file
ms.openlocfilehash: 7ad3e88d9bdd52c9dcba0bd52924eca14f995ed3
ms.sourcegitcommit: f4bd97855236f11020f968cfd5fbb0a4e84f9576
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/18/2020
ms.locfileid: "91146370"
---
当需要降低引入和查询之间的延迟时，请使用流式引入来加载数据。 流式引入操作会在 10 秒内完成，完成后数据立即可用于查询。 这种引入方法适合引入大量数据，如每秒数千条记录（分布在数千个表中）。 每个表接收的数据量相对较少，如每秒几条记录。

当每个表每小时引入的数据量超过 4 GB 时，请使用批量引入而不是流式引入。

若要详细了解各种引入方法，请参阅[数据引入概述](../ingest-data-overview.md)。
