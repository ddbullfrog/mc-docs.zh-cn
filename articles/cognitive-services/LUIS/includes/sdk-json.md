---
title: include 文件
description: include 文件
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 10/23/2020
ms.topic: include
ms.custom: include file, devx-track-js
ms.openlocfilehash: 3d342ed92f7e3257bdf5f8d4a0ec11d7f0d15575
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472973"
---
预测响应是一个 JSON 对象，其中包括意向和找到的所有实体。

```JSON
{
    "query": "I want two small pepperoni pizzas with more salsa",
    "prediction": {
        "topIntent": "OrderPizzaIntent",
        "intents": {
            "OrderPizzaIntent": {
                "score": 0.753606856
            },
            "None": {
                "score": 0.119097039
            }
        },
        "entities": {
            "Pizza order": [
                {
                    "Pizza": [
                        {
                            "Quantity": [
                                2
                            ],
                            "Type": [
                                "pepperoni"
                            ],
                            "Size": [
                                "small"
                            ],
                            "$instance": {
                                "Quantity": [
                                    {
                                        "type": "builtin.number",
                                        "text": "two",
                                        "startIndex": 7,
                                        "length": 3,
                                        "score": 0.968156934,
                                        "modelTypeId": 1,
                                        "modelType": "Entity Extractor",
                                        "recognitionSources": [
                                            "model"
                                        ]
                                    }
                                ],
                                "Type": [
                                    {
                                        "type": "Type",
                                        "text": "pepperoni",
                                        "startIndex": 17,
                                        "length": 9,
                                        "score": 0.9345611,
                                        "modelTypeId": 1,
                                        "modelType": "Entity Extractor",
                                        "recognitionSources": [
                                            "model"
                                        ]
                                    }
                                ],
                                "Size": [
                                    {
                                        "type": "Size",
                                        "text": "small",
                                        "startIndex": 11,
                                        "length": 5,
                                        "score": 0.9592077,
                                        "modelTypeId": 1,
                                        "modelType": "Entity Extractor",
                                        "recognitionSources": [
                                            "model"
                                        ]
                                    }
                                ]
                            }
                        }
                    ],
                    "Toppings": [
                        {
                            "Type": [
                                "salsa"
                            ],
                            "Quantity": [
                                "more"
                            ],
                            "$instance": {
                                "Type": [
                                    {
                                        "type": "Type",
                                        "text": "salsa",
                                        "startIndex": 44,
                                        "length": 5,
                                        "score": 0.7292897,
                                        "modelTypeId": 1,
                                        "modelType": "Entity Extractor",
                                        "recognitionSources": [
                                            "model"
                                        ]
                                    }
                                ],
                                "Quantity": [
                                    {
                                        "type": "Quantity",
                                        "text": "more",
                                        "startIndex": 39,
                                        "length": 4,
                                        "score": 0.9320932,
                                        "modelTypeId": 1,
                                        "modelType": "Entity Extractor",
                                        "recognitionSources": [
                                            "model"
                                        ]
                                    }
                                ]
                            }
                        }
                    ],
                    "$instance": {
                        "Pizza": [
                            {
                                "type": "Pizza",
                                "text": "two small pepperoni pizzas",
                                "startIndex": 7,
                                "length": 26,
                                "score": 0.812199831,
                                "modelTypeId": 1,
                                "modelType": "Entity Extractor",
                                "recognitionSources": [
                                    "model"
                                ]
                            }
                        ],
                        "Toppings": [
                            {
                                "type": "Toppings",
                                "text": "more salsa",
                                "startIndex": 39,
                                "length": 10,
                                "score": 0.7250252,
                                "modelTypeId": 1,
                                "modelType": "Entity Extractor",
                                "recognitionSources": [
                                    "model"
                                ]
                            }
                        ]
                    }
                }
            ],
            "$instance": {
                "Pizza order": [
                    {
                        "type": "Pizza order",
                        "text": "two small pepperoni pizzas with more salsa",
                        "startIndex": 7,
                        "length": 42,
                        "score": 0.769223332,
                        "modelTypeId": 1,
                        "modelType": "Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ]
            }
        }
    }
}
```

