---
title: include 文件
description: include 文件
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.date: 10/19/2020
ms.subservice: language-understanding
ms.topic: include
ms.custom: include file
ms.author: v-johya
ms.openlocfilehash: abe72d2a1b7a146829c1c514ddb834e412c6dc38
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472516"
---
示例言语采用特定格式。

`text` 字段包含示例话语的文本。 `intentName` 字段必须对应于 LUIS 应用中的现有意向名称。 `entityLabels` 字段是必填的。 如果不想标记任何实体，请提供一个空数组。

如果 entityLabels 数组不为空，则 `startCharIndex` 和 `endCharIndex` 需要标记 `entityName` 字段中引用的实体。 该索引从零开始。 如果标签的起始或结尾位于文本中的空白处，则添加话语的 API 调用将失败。

```JSON
[
  {
    "text": "order a pizza",
    "intentName": "ModifyOrder",
    "entityLabels": [
      {
        "entityName": "Order",
        "startCharIndex": 6,
        "endCharIndex": 12
      }
    ]
  },
  {
    "text": "order a large pepperoni pizza",
    "intentName": "ModifyOrder",
    "entityLabels": [
      {
        "entityName": "Order",
        "startCharIndex": 6,
        "endCharIndex": 28
      },
      {
        "entityName": "FullPizzaWithModifiers",
        "startCharIndex": 6,
        "endCharIndex": 28
      },
      {
        "entityName": "PizzaType",
        "startCharIndex": 14,
        "endCharIndex": 28
      },
      {
        "entityName": "Size",
        "startCharIndex": 8,
        "endCharIndex": 12
      }
    ]
  },
  {
    "text": "I want two large pepperoni pizzas on thin crust",
    "intentName": "ModifyOrder",
    "entityLabels": [
      {
        "entityName": "Order",
        "startCharIndex": 7,
        "endCharIndex": 46
      },
      {
        "entityName": "FullPizzaWithModifiers",
        "startCharIndex": 7,
        "endCharIndex": 46
      },
      {
        "entityName": "PizzaType",
        "startCharIndex": 17,
        "endCharIndex": 32
      },
      {
        "entityName": "Size",
        "startCharIndex": 11,
        "endCharIndex": 15
      },
      {
        "entityName": "Quantity",
        "startCharIndex": 7,
        "endCharIndex": 9
      },
      {
        "entityName": "Crust",
        "startCharIndex": 37,
        "endCharIndex": 46
      }
    ]
  }
]
```

