---
title: 容器存储库和映像
services: cognitive-services
author: aahill
manager: nitinme
description: 表示所有认知服务产品的容器注册表、存储库和映像名称的两个表。
ms.service: cognitive-services
ms.topic: include
ms.date: 08/07/2020
ms.author: v-johya
ms.openlocfilehash: 1d6198fb2e3f4c1b489c4ef406f3306c87c37d0c
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128282"
---
### <a name="container-repositories-and-images"></a>容器存储库和映像

下表是 Azure 认知服务提供的可用容器映像的列表。

#### <a name="generally-available"></a>正式发布 

Microsoft 容器注册表 (MCR) 同步发布了认知服务的所有正式发布的容器。 还可直接从 [Docker Hub](https://hub.docker.com/_/microsoft-azure-cognitive-services) 获取容器。

#### <a name="luis"></a>[LUIS](#tab/luis)

| LUIS 容器 | 容器注册表/存储库/映像名称 |
|--|--|
| LUIS | `mcr.microsoft.com/azure-cognitive-services/luis` |

有关详细信息，请参阅[如何运行和安装 LUIS 容器](../../LUIS/luis-container-howto.md)。

#### <a name="text-analytics"></a>[文本分析](#tab/text-analytics)

| 文本分析容器 | 容器注册表/存储库/映像名称 |
|--|--|
| 情绪分析 v3（英语） | `mcr.microsoft.com/azure-cognitive-services/sentiment:3.0-en` |
| 情绪分析 v3（西班牙语） | `mcr.microsoft.com/azure-cognitive-services/sentiment:3.0-es` |
| 情绪分析 v3（法语） | `mcr.microsoft.com/azure-cognitive-services/sentiment:3.0-fr` |
| 情绪分析 v3（意大利语） | `mcr.microsoft.com/azure-cognitive-services/sentiment:3.0-it` |
| 情绪分析 v3（德语） | `mcr.microsoft.com/azure-cognitive-services/sentiment:3.0-de` |
| 情绪分析 v3（简体中文） | `mcr.microsoft.com/azure-cognitive-services/sentiment:3.0-zh` |
| 情绪分析 v3（繁体中文） | `mcr.microsoft.com/azure-cognitive-services/sentiment:3.0-zht` |
| 情绪分析 v3（日语） | `mcr.microsoft.com/azure-cognitive-services/sentiment:3.0-ja` |
| 情绪分析 v3（葡萄牙语） | `mcr.microsoft.com/azure-cognitive-services/sentiment:3.0-pt` |
| 情绪分析 v3（荷兰语） | `mcr.microsoft.com/azure-cognitive-services/sentiment:3.0-nl` |

有关详细信息，请参阅[如何运行和安装文本分析容器](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md)。

---

#### <a name="public-ungated-preview-container-registry-mcrmicrosoftcom"></a>公共“非门控式”预览版（容器注册表：`mcr.microsoft.com`）

以下预览版容器现已公开提供。 Microsoft 容器注册表 (MCR) 同步发布了认知服务的所有公共可用的“非门控式”容器。 还可直接从 [Docker Hub](https://hub.docker.com/_/microsoft-azure-cognitive-services) 获取容器。

| 服务 | 容器 | 容器注册表/存储库/映像名称 |
|--|--|--|
| [文本分析](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md) | 关键短语提取 | `mcr.microsoft.com/azure-cognitive-services/keyphrase` |
| [文本分析](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md) | 语言检测 | `mcr.microsoft.com/azure-cognitive-services/language` |

#### <a name="public-gated-preview-container-registry-containerpreviewazurecrio"></a>公共“门控式”预览版（容器注册表：`containerpreview.azurecr.io`）

以下门控式预览版容器托管在容器预览版注册表上，需要应用程序才能访问。 有关详细信息，请参阅以下容器文章。

| 服务 | 容器 | 容器注册表/存储库/映像名称 |
|--|--|--|
| [人脸](../../face/face-how-to-install-containers.md) | 人脸 | `containerpreview.azurecr.io/microsoft/cognitive-services-face` |
| [运行状况文本分析](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=health) | 运行状况文本分析 | `containerpreview.azurecr.io/microsoft/cognitive-services-healthcare` |

