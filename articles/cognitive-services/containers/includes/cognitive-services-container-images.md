---
title: 容器存储库和映像
services: cognitive-services
author: Johnnytechn
manager: nitinme
description: 表示所有认知服务产品的容器注册表、存储库和映像名称的两个表。
ms.service: cognitive-services
ms.topic: include
ms.date: 10/26/2020
ms.author: v-johya
ms.openlocfilehash: 19de0d70c4d6cbf7ea3cc2880a30e721751052d1
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104617"
---
### <a name="container-repositories-and-images"></a>容器存储库和映像

下表是 Azure 认知服务提供的可用容器映像的列表。 有关所有可用容器映像名称及其可用标记的完整列表，请参阅[认知服务容器映像标记](../container-image-tags.md)。 

#### <a name="generally-available"></a>正式发布 

Microsoft 容器注册表 (MCR) 同步发布了认知服务的所有正式发布的容器。 还可直接从 [Docker Hub](https://hub.docker.com/_/microsoft-azure-cognitive-services) 获取容器。

**LUIS**

| 容器 | 容器注册表/存储库/映像名称 |
|--|--|
| LUIS | `mcr.microsoft.com/azure-cognitive-services/language/luis` |

有关详细信息，请参阅[如何运行和安装 LUIS 容器](../../LUIS/luis-container-howto.md)。

**文本分析**

| 容器 | 容器注册表/存储库/映像名称 |
|--|--|
| 情绪分析 v3（英语） | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-en` |
| 情绪分析 v3（西班牙语） | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-es` |
| 情绪分析 v3（法语） | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-fr` |
| 情绪分析 v3（意大利语） | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-it` |
| 情绪分析 v3（德语） | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-de` |
| 情绪分析 v3（简体中文） | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-zh` |
| 情绪分析 v3（繁体中文） | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-zht` |
| 情绪分析 v3（日语） | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-ja` |
| 情绪分析 v3（葡萄牙语） | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-pt` |
| 情绪分析 v3（荷兰语） | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-nl` |

有关详细信息，请参阅[如何运行和安装文本分析容器](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md)。

<!--Not available in MC: **Anomaly Detector** -->
#### <a name="ungated-preview"></a>“非门控式”预览版 

以下预览版容器现已公开提供。 Microsoft 容器注册表 (MCR) 同步发布了认知服务的所有公共可用的“非门控式”容器。 还可直接从 [Docker Hub](https://hub.docker.com/_/microsoft-azure-cognitive-services) 获取容器。

| 服务 | 容器 | 容器注册表/存储库/映像名称 |
|--|--|--|
| [文本分析](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md) | 关键短语提取 | `mcr.microsoft.com/azure-cognitive-services/textanalytics/keyphrase` |
| [文本分析](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md) | 语言检测 | `mcr.microsoft.com/azure-cognitive-services/textanalytics/language` |


#### <a name="gated-preview"></a>“门控式”预览版

以前，门控式预览版容器托管在 `containerpreview.azurecr.io` 存储库中。 从 2020 年 9 月 22 日开始，这些容器（“运行状况文本分析”除外）托管在 Microsoft 容器注册表 (MCR) 中，下载它们不需要使用 docker login 命令。 若要使用容器，你将需要：

1. 使用 Azure 订阅 ID 和用户方案填写[请求表单](https://aka.ms/csgate)。 
2. 获得批准后，从 MCR 下载容器。 
3. 使用相应 Azure 资源中的密钥和终结点在运行时进行容器身份验证。 

| 服务 | 容器 | 容器注册表/存储库/映像名称 |
|--|--|--|
| [运行状况文本分析](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=health) | 运行状况文本分析 | `containerpreview.azurecr.io/microsoft/cognitive-services-healthcare` |


