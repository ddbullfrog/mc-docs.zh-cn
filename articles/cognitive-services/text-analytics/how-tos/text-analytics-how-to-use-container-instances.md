---
title: 运行 Azure 容器实例 - 文本分析
titleSuffix: Azure Cognitive Services
description: 将文本分析容器部署到 Azure 容器实例，并在 Web 浏览器中对其进行测试。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 10/26/2020
ms.author: v-johya
ms.openlocfilehash: d3b6ccea2e17647ab3740e1a4118e63560126c83
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103924"
---
# <a name="deploy-a-text-analytics-container-to-azure-container-instances"></a>将文本分析容器部署到 Azure 容器实例

了解如何将认知服务[文本分析][install-and-run-containers]容器部署到 Azure [容器实例][container-instances]。 该过程演示了创建文本分析资源的方法、创建关联的情绪分析映像的方法，以及在浏览器中练习前两项的相关业务流程的功能。 使用容器可以将开发人员的注意力从管理基础结构转移到应用程序开发上。

## <a name="prerequisites"></a>必备条件

* 使用 Azure 订阅。 如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/details/cognitive-services)。

[!INCLUDE [Create a Cognitive Services Text Analytics resource](../includes/create-text-analytics-resource.md)]

[!INCLUDE [Create a Text Analytics Containers on Azure Container Instances](../../containers/includes/create-container-instances-resource.md)]

#### <a name="key-phrase-extraction"></a>[关键短语提取](#tab/keyphrase)

[!INCLUDE [Verify the Key Phrase Extraction container instance](../includes/verify-key-phrase-extraction-container.md)]

#### <a name="language-detection"></a>[语言检测](#tab/language)

[!INCLUDE [Verify the Language Detection container instance](../includes/verify-language-detection-container.md)]

#### <a name="sentiment-analysis"></a>[情绪分析](#tab/sentiment)

[!INCLUDE [Verify the Sentiment Analysis container instance](../includes/verify-sentiment-analysis-container.md)]

#### <a name="text-analytics-for-health"></a>[运行状况文本分析](#tab/health)

[!INCLUDE [Verify the health container instance](../includes/verify-health-container.md)]

***

## <a name="next-steps"></a>后续步骤 

* 使用更多[认知服务容器](../../cognitive-services-container-support.md)
* 使用[文本分析连接服务](../vs-text-connected-service.md)

[install-and-run-containers]: ./text-analytics-how-to-install-containers.md
[container-instances]: /container-instances

