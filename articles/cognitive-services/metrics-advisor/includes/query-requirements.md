---
title: 查询要求
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: include
ms.date: 10/22/2020
ms.author: v-johya
ms.openlocfilehash: 601a4963bc73daba0d682c019cd2b7b804a7c6fb
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472868"
---
在查询中，使用 `@StartTime` 参数获取单一时间戳的指标数据。 指标顾问将在运行查询时将参数替换为 `yyyy-MM-ddTHH:mm:ss` 格式字符串。

> [!IMPORTANT]
> 对于每个维度组合，查询应在每个时间戳处最多返回一条记录。 查询返回的所有记录必须具有相同的时间戳。 指标顾问将针对每个时间戳运行此查询以引入数据。 有关详细信息和示例，请参阅[查询常见问题解答部分](../faq.md#how-do-i-write-a-valid-query-for-ingesting-my-data)。 

