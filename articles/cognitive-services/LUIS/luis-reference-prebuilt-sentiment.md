---
title: 情绪分析 - LUIS
titleSuffix: Azure Cognitive Services
description: 如果配置了情绪分析，LUIS json 响应会包含情绪分析内容。
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 10/19/2020
ms.author: v-johya
origin.date: 10/22/2019
ms.openlocfilehash: 0de5269dc359d007a6f175fd9efae4fa55265b68
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472370"
---
# <a name="sentiment-analysis"></a>情绪分析
如果配置了情绪分析，LUIS json 响应会包含情绪分析内容。 请在[文本分析](/cognitive-services/text-analytics/)文档中详细了解情绪分析。

LUIS 使用文本分析 V2。 

## <a name="resolution-for-sentiment"></a>情绪解决方法

情绪数据是一个介于 0 到 1 之间的分数，指示数据的正面情绪（分数接近 1）或负面情绪（分数接近 0）。

#### <a name="english-language"></a>[英语](#tab/english)

当区域性为 `en-us` 时，响应为：

```JSON
"sentimentAnalysis": {
  "label": "positive",
  "score": 0.9163064
}
```

#### <a name="other-languages"></a>[其他语言](#tab/other-languages)

对于所有其他区域性，响应为：

```JSON
"sentimentAnalysis": {
  "score": 0.9163064
}
```
* * *

## <a name="next-steps"></a>后续步骤

详细了解 [V3 预测终结点](luis-migration-api-v3.md)。


