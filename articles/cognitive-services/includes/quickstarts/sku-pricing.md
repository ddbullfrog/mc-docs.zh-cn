---
title: 认知服务 SKU 和定价
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 10/28/2020
ms.author: v-johya
ms.openlocfilehash: 076ecb75f20bc96f28029d2b97ed9f7be6388b89
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106474"
---
请参阅下面的 SKU 和定价信息列表。 

#### <a name="multi-service"></a>多服务

| 服务                    | 种类                      |
|----------------------------|---------------------------|
| 多个服务。 有关详细信息，请参阅[定价](https://www.azure.cn/pricing/details/cognitive-services/)页。            | `CognitiveServices`     |


#### <a name="vision"></a>影像

| 服务                    | 种类                      |
|----------------------------|---------------------------|
| 计算机视觉            | `ComputerVision`          |
| 人脸                       | `Face`                    |

#### <a name="speech"></a>语音

| 服务            | 种类                 |
|--------------------|----------------------|
| 语音服务    | `SpeechServices`     |
| 语音识别 | `SpeakerRecognition` |

#### <a name="language"></a>语言

| 服务            | 种类                |
|--------------------|---------------------|
| LUIS               | `LUIS`              |
| 文本分析     | `TextAnalytics`     |
| 文本翻译   | `TextTranslation`   |

#### <a name="decision"></a>决策

| 服务           | 种类               |
|-------------------|--------------------|
| 内容审查器 | `ContentModerator` |


#### <a name="pricing-tiers-and-billing"></a>定价层和计费

定价层（以及你收到的账单金额）基于你使用身份验证信息发送的事务数。 每个定价层指定：
* 每秒允许的最大事务数 (TPS)。
* 在定价层中启用的服务功能。
* 预定义事务数的成本。 根据[定价详细信息](https://www.azure.cn/pricing/details/cognitive-services/)中为服务所指定的内容，超过此数字将导致额外费用。

> [!NOTE]
> 许多认知服务都有一个免费层，供你试用该服务。 若要使用免费层，请使用 `F0` 作为资源的 SKU。

