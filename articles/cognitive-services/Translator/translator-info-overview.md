---
title: 翻译器服务
titlesuffix: Azure Cognitive Services
description: 将“翻译”集成到应用程序、网站、工具和其他解决方案中，提供多语言用户体验。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: overview
origin.date: 06/26/2020
ms.date: 10/22/2020
ms.author: v-johya
ms.custom: cog-serv-seo-aug-2020
keywords: translator, 文本翻译, 机器翻译, 翻译服务
ms.openlocfilehash: 5f8c99a992365aaa1c719d865dc4768a79d7765d
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472688"
---
# <a name="what-is-the-translator-service"></a>什么是 Translator 服务？

Translator 是一种基于云的机器翻译服务，是用于构建智能应用的 [Azure 认知服务](https://docs.azure.cn/cognitive-services/)系列认知 API 的一部分。 “翻译”可以轻松地集成到应用程序、网站、工具和解决方案中。 通过它，你可以添加[超过 70 种语言](languages.md)的多语言用户体验，并且可以在具有任何操作系统的任何硬件平台上将其应用于文本翻译。

## <a name="about-microsoft-translator"></a>关于 Microsoft Translator

“翻译”为各种 Microsoft 产品和服务提供支持，并已在全球数千家企业的应用程序和工作流中使用，使其内容可传播到全球的受众。

由 Translator 提供支持的语音翻译也可通过 [Azure 语音服务](/cognitive-services/speech-service/)获取。 它将语音翻译 API 和自定义语音服务的功能组合成了一项统一的可完全自定义的服务。 

## <a name="language-support"></a>语言支持

Translator 为文本翻译、音译、语言检测和字典提供多语言支持。 请参阅[语言支持](language-support.md)以获取完整的列表，或者通过 [REST API](/cognitive-services/translator/reference/v3-0-languages) 以编程方式访问列表。  

## <a name="microsoft-translator-neural-machine-translation"></a>Microsoft Translator 神经机器翻译

神经机器翻译 (NMT) 是采用 AI 的高质量机器翻译的新标准。 它代替的是旧式统计机器翻译 (SMT) 技术，该技术在 2010-2020 中期的几年中达到了质量顶峰。

与 SMT 相比，NMT 不仅能够从原始翻译质量评分的立场提供更好的翻译，而且听起来更流畅、更类似于人类。 这种流畅性的主要原因在于 NMT 使用一个句子的完整语境来翻译单词。 SMT 仅考虑每个单词前面和后面几个单词的直接语境。

NMT 模型是该 API 的核心，对最终用户不可见。 唯一明显的区别是改进的翻译质量，尤其是针对中文、日语和阿拉伯语等语言。

详细了解 [NMT 的工作原理](https://www.microsoft.com/translator/mt.aspx#nnt)

<!-- Language customization not support in Azure-->

## <a name="next-steps"></a>后续步骤

- [注册](translator-text-how-to-signup.md)访问密钥。
- [API 参考](/cognitive-services/Translator/reference/v3-0-reference)提供了 API 的技术文档。
- [定价详细信息](https://www.azure.cn/pricing/details/cognitive-services)

<!-- Update_Description: wording update -->