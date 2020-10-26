---
title: URL 预生成实体 - LUIS
titleSuffix: Azure Cognitive Services
description: 本文包含语言理解 (LUIS) 中的 URL 预构建实体信息。
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.author: v-johya
ms.topic: reference
ms.date: 10/19/2020
origin.date: 10/04/2019
ms.openlocfilehash: dda21076473c5fac2a435fb4a82a46fb8199330b
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472367"
---
# <a name="url-prebuilt-entity-for-a-luis-app"></a>LUIS 应用的 URL 预生成实体
URL 实体提取带域名或 IP 地址的 URL。 此实体已定型，因此不需要将包含 URL 的示例陈述添加到应用程序。 仅在 `en-us` 区域性中支持 URL 实体。

## <a name="types-of-urls"></a>URL 类型
URL 托管在 [Recognizers-text](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/Base-URL.yaml) GitHub 存储库

## <a name="resolution-for-prebuilt-url-entity"></a>预构建 URL 实体解析

查询返回以下实体对象：

`https://luis.azure.cn is a great cognitive services example of artificial intelligence`

#### <a name="v3-response"></a>[V3 响应](#tab/V3)

以下 JSON 的 `verbose` 参数设置为 `false`：

```json
"entities": {
    "url": [
        "https://luis.azure.cn"
    ]
}
```
#### <a name="v3-verbose-response"></a>[V3 详细响应](#tab/V3-verbose)

以下 JSON 的 `verbose` 参数设置为 `true`：

```json
"entities": {
    "url": [
        "https://luis.azure.cn"
    ],
    "$instance": {
        "url": [
            {
                "type": "builtin.url",
                "text": "https://luis.azure.cn",
                "startIndex": 0,
                "length": 17,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```
#### <a name="v2-response"></a>[V2 响应](#tab/V2)

以下示例显示了 https://luis.azure.cn 的解析是人工智能的一个良好认知服务示例

```json
"entities": [
    {
        "entity": "https://luis.azure.cn",
        "type": "builtin.url",
        "startIndex": 0,
        "endIndex": 17
    }
]
```

* * *

## <a name="next-steps"></a>后续步骤

详细了解 [V3 预测终结点](luis-migration-api-v3.md)。

了解[序号](luis-reference-prebuilt-ordinal.md)、[数字](luis-reference-prebuilt-number.md)和[温度](luis-reference-prebuilt-temperature.md)实体。

