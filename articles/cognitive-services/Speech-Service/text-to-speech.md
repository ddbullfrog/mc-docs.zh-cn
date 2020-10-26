---
title: 文本转语音概述 - 语音服务
titleSuffix: Azure Cognitive Services
description: 语音服务中的文本转语音功能可让应用程序、工具或设备将文本转换为类似于人类的自然合成语音。 本文概述了文本转语音服务的优势和功能。
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
origin.date: 09/01/2020
ms.date: 10/16/2020
ms.author: v-tawe
ms.custom: cog-serv-seo-aug-2020
keywords: 文本转语音
ms.openlocfilehash: 75ed2d49bb95c46bf1d5c10cccf92a1b23c2a23d
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128383"
---
# <a name="what-is-text-to-speech"></a>什么是文本转语音？

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

在本概述中，你将了解文本转语音服务的优势和功能，该服务使你的应用程序、工具或设备可以将文本转换为类似于人的合成语音。 从标准和神经语音中选择，或创建产品或品牌特有的自定义语音。 40 多种标准语音已在 45 种以上的语言和区域设置中提供，10 种神经语音已在一组精选的语言和区域设置中提供。 有关支持的语音、语言和区域设置的完整列表，请参阅[支持的语言](language-support.md#text-to-speech)。

<!-- > [!NOTE] -->
<!-- > Bing Speech was decommissioned on October 15, 2019. If your applications, tools, or products are using the Bing Speech APIs or Custom Speech, we've created guides to help you migrate to the Speech service. -->
<!-- > - [Migrate from Bing Speech to the Speech service](how-to-migrate-from-bing-speech.md) -->

## <a name="core-features"></a>核心功能

* 语音合成 - 使用[语音 SDK](quickstarts/text-to-speech-audio-file.md) 或 [REST API](rest-text-to-speech.md) 通过标准语音和神经语音将文本转换为语音。

* 长音频的异步合成 - 使用[长音频 API](long-audio-api.md) 异步合成 10 分钟以上的文本转语音文件（例如有声书籍或讲座）。 不同于使用语音 SDK 或语音转文本 REST API 执行的合成，响应不会实时返回。 预期会异步发送请求，以轮询的方式获取响应，并会下载合成音频（在服务提供该音频的情况下）。 仅支持神经语音。

* 标准语音 - 使用统计参数合成和/或串联合成技术创建。 这些语音的辨识度很高，且听起来非常自然。 你可以轻松地让应用程序使用多种语音选项以 10 种以上的语言讲述。 这些声音提供较高的发音准确度，支持缩写、缩略词扩展、日期/时间解释、多音字等。 有关标准语音的完整列表，请参阅[支持的语言](language-support.md#text-to-speech)。

* 神经语音 - 深层神经网络用于克服有关口语中的重读和语调的传统语音合成限制。 韵律预测和语音合成以同步方式执行，使输出听起来更流畅且自然。 使用神经语音可使得与聊天机器人和语音助手的交流更加自然且富有吸引力、将数字文本（如电子书）转换为有声读物以及增强车载导航系统。 神经语音可以生成类人的自然韵律和清晰的字词发音，当你在与 AI 系统交互时，它可以显著减轻听力疲劳。 有关神经语音的完整列表，请参阅[支持的语言](language-support.md#text-to-speech)。

* 语音合成标记语言 (SSML) - 一种基于 XML 的标记语言，用于自定义语音转文本输出。 使用 SSML，你可以调整音调、添加暂停、改进发音、提高或降低语速、增加或减少音量，以及将多个语音赋予单个文档。 请参阅 [SSML](speech-synthesis-markup.md)。

## <a name="get-started"></a>入门

请参阅[快速入门](get-started-text-to-speech.md)以开始使用文本转语音。 文本转语音服务通过[语音 SDK](speech-sdk.md)、[REST API](rest-text-to-speech.md) 和[语音 CLI](spx-overview.md) 提供

## <a name="sample-code"></a>代码示例

GitHub 上提供了文本转语音的示例代码。 这些示例涵盖了最流行编程语言的文本转语音转换。

- [文本转语音示例 (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
- [文本转语音示例 (REST)](https://github.com/Azure-Samples/Cognitive-Speech-TTS)

<!-- ## Customization  -->

## <a name="pricing-note"></a>定价说明

使用文本转语音服务时，需按照转换为语音的每个字符（包括标点）付费。 尽管 SSML 文档本身不计费，但用于调整文本转语音方式的可选元素（例如音素和音节）将算作计费字符。 下面列出了计费的内容：

- 在请求的 SSML 正文中传递给文本转语音服务的文本
- 请求正文的文本字段中所有 SSML 格式的标记，`<speak>` 和 `<voice>` 标记除外
- 字母、标点、空格、制表符、标记和所有空白字符
- Unicode 中定义的每个码位

有关详细信息，请参阅[定价](https://www.azure.cn/pricing/details/cognitive-services/)。

> [!IMPORTANT]
> 每个中文、日语和韩语字符算作两个计费字符。

## <a name="reference-docs"></a>参考文档

- [语音 SDK](speech-sdk.md)
- [REST API：文本转语音](rest-text-to-speech.md)

## <a name="next-steps"></a>后续步骤

- [获取免费语音服务订阅](overview.md#try-the-speech-service-for-free)
- [获取语音 SDK](speech-sdk.md)
