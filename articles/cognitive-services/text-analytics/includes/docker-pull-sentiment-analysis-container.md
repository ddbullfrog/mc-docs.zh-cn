---
title: 适用于情绪分析容器的 Docker pull
titleSuffix: Azure Cognitive Services
description: 适用于情绪分析容器的 Docker pull 命令
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 08/07/2020
ms.author: v-johya
ms.openlocfilehash: a3fae63c02c0cad2cab210f8b7e6208f5edd28be
ms.sourcegitcommit: caa18677adb51b5321ad32ae62afcf92ac00b40b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/08/2020
ms.locfileid: "92211343"
---
#### <a name="docker-pull-for-the-sentiment-analysis-v3-container"></a>适用于情绪分析 v3 容器的 Docker pull

情绪分析容器 v3 容器以多种语言提供。 若要下载英文版容器，请使用以下命令。 

```
docker pull mcr.microsoft.com/azure-cognitive-services/sentiment:3.0-en
```

若要下载其他语言版的容器，请将 `en` 替换为以下语言代码之一。 

| 文本分析容器 | 语言代码 |
|--|--|
| 英语 | `en` |
| 西班牙语 | `es` |
| 法语 | `fr` |
| 意大利语 | `it` |
| 德语 | `de` |
| 简体中文 | `zh` |
| 繁体中文 | `zht` |
| 日语 | `ja` |
| 葡萄牙语 | `pt` |
| 荷兰语 | `nl` |

有关文本分析容器可用标记的完整说明，请查阅 [Docker 中心](https://go.microsoft.com/fwlink/?linkid=2018654)。

