---
title: 认知服务容器映像标记
titleSuffix: Azure Cognitive Services
description: 所有认知服务容器映像标记的全面列表。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: reference
ms.date: 10/26/2020
ms.author: v-johya
ms.openlocfilehash: c15e74abf10f6fa3e28604d3495b91a84ad0fcc2
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106808"
---
# <a name="azure-cognitive-services-container-image-tags"></a>Azure 认知服务容器映像标记

Azure 认知服务提供了许多容器映像。 容器注册表和相应的存储库因容器映像而异。 每个容器映像名称都提供多个标记。 容器映像标记是对容器映像进行版本控制的机制。 本文旨在作为综合性的参考资料使用，其中列出了所有认知服务容器映像及其可用标记。

> [!TIP]
> 在使用 [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) 时，请密切注意容器注册表、存储库、容器映像名称和相应标记的大小写，因为它们是区分大小写的。

## <a name="computer-vision"></a>计算机视觉

可在 `containerpreview.azurecr.io` 容器注册表中找到“计算机视觉阅读 OCR”容器映像。 它驻留在 `microsoft` 存储库中，名为 `cognitive-services-read`。 完全限定的容器映像名称为 `containerpreview.azurecr.io/microsoft/cognitive-services-read`。

此容器映像提供了以下标记：

| 映像标记                    | 说明 |
|-------------------------------|:------|
| `latest ( (2.0.013250001-amd64-preview)` | • 进一步降低容器的内存使用率。 |
|                                          | • 对于多 Pod 设置，需要外部缓存。 例如，设置 Redis 以用于缓存。 |
|                                          | • 修复了在设置 Redis 缓存并且 ResultExpirationPeriod=0 时缺失结果的问题。  |
|                                          | • 删除了请求正文大小不得超过 26MB 的限制。 容器现在可以接受大于 26MB 的文件。  |
|                                          | • 为控制台日志记录添加了时间戳和内部版本。  |
| `1.1.013050001-amd64-preview`            | * 添加了 ReadEngineConfig:ResultExpirationPeriod 容器初始化配置，以指定系统何时应清除识别结果。 |
|                                          | 该设置以小时为单位，默认值为 48 小时。   |
|                                          |   该设置可减少用于存储结果的内存使用量，尤其是在使用容器内存中存储时。  |
|                                          |    * 示例 1. ReadEngineConfig:ResultExpirationPeriod=1，系统将在进程结束后 1 小时清除识别结果。   |
|                                          |    * 示例 2. ReadEngineConfig:ResultExpirationPeriod=0，系统将在检索结果之后清除识别结果。  |
|                                          | 修复了在无效图像格式传入系统时出现的 500 内部服务器错误。 现在，它将会返回 400 错误：   |
|                                          | `{`  |
|                                          | `"error": {`  |
|                                          |      `"code": "InvalidImageSize",`  |
|                                          |      `"message": "Image must be between 1024 and 209715200 bytes."`  |
|                                          |          `}`  |
|                                          | `}`  |
| `1.1.011580001-amd64-preview` |       |
| `1.1.009920003-amd64-preview` |       |
| `1.1.009910003-amd64-preview` |       |

## <a name="language-understanding-luis"></a>语言理解 (LUIS)

在 `mcr.microsoft.com` 容器注册表联合项中可以找到 [LUIS][lu-containers] 容器映像。 该映像驻留在 `azure-cognitive-services` 存储库中，名为 `luis`。 完全限定的容器映像名称为 `mcr.microsoft.com/azure-cognitive-services/luis`。

此容器映像提供了以下标记：

| 映像标记                    | 说明 |
|-------------------------------|:------|
| `latest`                      |       |
| `1.1.010330004-amd64-preview` |       |
| `1.1.009301-amd64-preview`    |       |
| `1.1.008710001-amd64-preview` |       |
| `1.1.008510001-amd64-preview` |       |
| `1.1.008010002-amd64-preview` |       |
| `1.1.007750002-amd64-preview` |       |
| `1.1.007360001-amd64-preview` |       |
| `1.1.007020001-amd64-preview` |       |


## <a name="neural-text-to-speech"></a>神经文本转语音

在 `containerpreview.azurecr.io` 容器注册表中可以找到 [神经文本转语音][sp-ntts] 容器映像。 该映像驻留在 `microsoft` 存储库中，名为 `cognitive-services-neural-text-to-speech`。 完全限定的容器映像名称为 `containerpreview.azurecr.io/microsoft/cognitive-services-neural-text-to-speech`。

此容器映像提供了以下标记：

| 映像标记                                  | 说明                                                                      |
|---------------------------------------------|:---------------------------------------------------------------------------|
| `latest`                                    | 具有 `en-US` 区域设置和 `en-US-AriaNeural` 语音的容器映像。      |
| `1.2.0-amd64-de-de-katjaneural-preview`     | 具有 `de-DE` 区域设置和 `de-DE-KatjaNeural` 语音的容器映像。     |
| `1.2.0-amd64-en-au-natashaneural-preview`   | 具有 `en-AU` 区域设置和 `en-AU-NatashaNeural` 语音的容器映像。   |
| `1.2.0-amd64-en-ca-claraneural-preview`     | 具有 `en-CA` 区域设置和 `en-CA-ClaraNeural` 语音的容器映像。     |
| `1.2.0-amd64-en-gb-libbyneural-preview`     | 具有 `en-GB` 区域设置和 `en-GB-LibbyNeural` 语音的容器映像。     |
| `1.2.0-amd64-en-gb-mianeural-preview`       | 具有 `en-GB` 区域设置和 `en-GB-MiaNeural` 语音的容器映像。       |
| `1.2.0-amd64-en-us-arianeural-preview`      | 具有 `en-US` 区域设置和 `en-US-AriaNeural` 语音的容器映像。      |
| `1.2.0-amd64-en-us-guyneural-preview`       | 具有 `en-US` 区域设置和 `en-US-GuyNeural` 语音的容器映像。       |
| `1.2.0-amd64-es-es-elviraneural-preview`    | 具有 `es-ES` 区域设置和 `es-ES-ElviraNeural` 语音的容器映像。    |
| `1.2.0-amd64-es-mx-dalianeural-preview`     | 具有 `es-MX` 区域设置和 `es-MX-DaliaNeural` 语音的容器映像。     |
| `1.2.0-amd64-fr-ca-sylvieneural-preview`    | 具有 `fr-CA` 区域设置和 `fr-CA-SylvieNeural` 语音的容器映像。    |
| `1.2.0-amd64-fr-fr-deniseneural-preview`    | 具有 `fr-FR` 区域设置和 `fr-FR-DeniseNeural` 语音的容器映像。    |
| `1.2.0-amd64-it-it-elsaneural-preview`      | 具有 `it-IT` 区域设置和 `it-IT-ElsaNeural` 语音的容器映像。      |
| `1.2.0-amd64-ja-jp-nanamineural-preview`    | 具有 `ja-JP` 区域设置和 `ja-JP-NanamiNeural` 语音的容器映像。    |
| `1.2.0-amd64-ko-kr-sunhineural-preview`     | 具有 `ko-KR` 区域设置和 `ko-KR-SunHiNeural` 语音的容器映像。     |
| `1.2.0-amd64-pt-br-franciscaneural-preview` | 具有 `pt-BR` 区域设置和 `pt-BR-FranciscaNeural` 语音的容器映像。 |
| `1.2.0-amd64-zh-cn-xiaoxiaoneural-preview`  | 具有 `zh-cn` 区域设置和 `zh-cn-XiaoxiaoNeural` 语音的容器映像。  |

## <a name="key-phrase-extraction"></a>关键短语提取

在 `mcr.microsoft.com` 容器注册表联合项中可以找到[关键短语提取][ta-kp]容器映像。 该映像驻留在 `azure-cognitive-services` 存储库中，名为 `keyphrase`。 完全限定的容器映像名称为 `mcr.microsoft.com/azure-cognitive-services/keyphrase`。

此容器映像提供了以下标记：

| 映像标记                    | 说明 |
|-------------------------------|:------|
| `latest`                      |       |
| `1.1.009301-amd64-preview`    |       |
| `1.1.008510001-amd64-preview` |       |
| `1.1.007750002-amd64-preview` |       |
| `1.1.007360001-amd64-preview` |       |
| `1.1.006770001-amd64-preview` |       |

## <a name="language-detection"></a>语言检测

在 `mcr.microsoft.com` 容器注册表联合项中可以找到[语言检测][ta-la]容器映像。 该映像驻留在 `azure-cognitive-services` 存储库中，名为 `language`。 完全限定的容器映像名称为 `mcr.microsoft.com/azure-cognitive-services/language`。

此容器映像提供了以下标记：

| 映像标记                    | 说明 |
|-------------------------------|:------|
| `latest`                      |       |
| `1.1.009301-amd64-preview`    |       |
| `1.1.008510001-amd64-preview` |       |
| `1.1.007750002-amd64-preview` |       |
| `1.1.007360001-amd64-preview` |       |
| `1.1.006770001-amd64-preview` |       |

## <a name="sentiment-analysis"></a>情绪分析

在 `mcr.microsoft.com` 容器注册表联合项中可以找到[情绪分析][ta-se]容器映像。 该映像驻留在 `azure-cognitive-services` 存储库中，名为 `sentiment`。 完全限定的容器映像名称为 `mcr.microsoft.com/azure-cognitive-services/sentiment`。

此容器映像提供了以下标记：

| 映像标记 | 说明                                         |
|------------|:----------------------------------------------|
| `latest`   |                                               |
| `3.0-en`   | 情绪分析 v3（英语）               |
| `3.0-es`   | 情绪分析 v3（西班牙语）               |
| `3.0-fr`   | 情绪分析 v3（法语）                |
| `3.0-it`   | 情绪分析 v3（意大利语）               |
| `3.0-de`   | 情绪分析 v3（德语）                |
| `3.0-zh`   | 情绪分析 v3（简体中文）  |
| `3.0-zht`  | 情绪分析 v3（繁体中文） |
| `3.0-ja`   | 情绪分析 v3（日语）              |
| `3.0-pt`   | 情绪分析 v3（葡萄牙语）            |
| `3.0-nl`   | 情绪分析 v3（荷兰语）                 |
| `1.1.009301-amd64-preview`    | 情绪分析 v2      |
| `1.1.008510001-amd64-preview` |       |
| `1.1.007750002-amd64-preview` |       |
| `1.1.007360001-amd64-preview` |       |
| `1.1.006770001-amd64-preview` |       |

[lu-containers]: ../luis/luis-container-howto.md
[ta-kp]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=keyphrase
[ta-la]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=language
[ta-se]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=sentiment

