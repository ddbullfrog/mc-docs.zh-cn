---
title: 容器支持
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 08/07/2020
ms.author: v-johya
ms.openlocfilehash: 97c7c8d3cb4726902b76ce156330ab0f76f9a80d
ms.sourcegitcommit: caa18677adb51b5321ad32ae62afcf92ac00b40b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/08/2020
ms.locfileid: "92211368"
---
## <a name="create-an-face-resource"></a>创建人脸资源

1. 登录到 [Azure 门户](https://portal.azure.cn)
1. 单击“[创建人脸](https://portal.azure.cn/#create/Microsoft.CognitiveServicesFace)资源”。
1. 输入所有必需的设置：

    |设置|Value|
    |--|--|
    |名称|所需名称（2-64 个字符）|
    |订阅|选择相应的订阅|
    |位置|选择附近任何可用的位置|
    |定价层|`F0` - 最小定价层|
    |资源组|选择可用的资源组|

1. 单击“创建”  并等待创建资源。 创建资源后，导航到“资源”页
1. 收集配置的 `endpoint` 和 API 密钥：

    |门户中的“资源”选项卡|设置|“值”|
    |--|--|--|
    |**概述**|端点|复制终结点。 它看起来类似于 `https://face.cognitiveservices.azure.cn/face/v1.0`|
    |**“键”**|API 密钥|复制两个密钥中的 1 个。 它是一个由 32 个字母数字组成的字符串（不包含空格或短划线），即 `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`。|

