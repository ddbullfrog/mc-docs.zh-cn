---
title: 什么是语音服务？
titleSuffix: Azure Cognitive Services
description: 语音服务在单个 Azure 订阅中统合了语音转文本、文本转语音以及语音翻译功能。 使用语音 SDK、语音设备 SDK 或 REST API 在应用程序、工具和设备中添加语音。
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: overview
origin.date: 08/20/2020
ms.date: 10/16/2020
ms.author: v-tawe
ms.openlocfilehash: 3142a57d35ad1ce50ec3df09187c383f436e212f
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92127685"
---
# <a name="what-is-the-speech-service"></a>什么是语音服务？

语音服务在单个 Azure 订阅中统合了语音转文本、文本转语音以及语音翻译功能。 使用[语音 CLI](spx-overview.md)、[语音 SDK](speech-sdk-reference.md)、[语音设备 SDK](speech-devices-sdk-android-quickstart.md)、[Speech Studio](https://speech.azure.cn/) 或 [REST API](rest-apis.md) 可以轻松在应用程序、工具和设备中启用语音。

<!-- > [!IMPORTANT] -->
<!-- > The Speech service has replaced Bing Speech API and Translator Speech. See _How-to guides > Migration_ for migration instructions. -->

以下功能是语音服务的一部分。 请使用下表中的链接详细了解每项功能的常见用例或浏览 API 参考信息。

| 服务 | 功能 | 说明 | SDK 中 IsInRole 中的声明 | REST |
|---------|---------|-------------|-----|------|
| [语音转文本](speech-to-text.md) | 实时语音转文本 | 语音转文本可将音频流或本地文件实时转录或翻译为文本，应用程序、工具或设备可以使用或显示这些文本。 结合[语言理解 (LUIS)](https://docs.azure.cn/cognitive-services/luis/) 使用语音转文本可以从听录的语音中派生用户意向，以及处理语音命令。 | [是](https://docs.azure.cn/cognitive-services/speech-service/speech-sdk-reference) | [是](https://docs.azure.cn/cognitive-services/speech-service/rest-apis) |
| | [批量语音转文本](batch-transcription.md) | 批量语音转文本支持对 Azure Blob 存储中存储的大量语音音频数据进行异步语音到文本转录。 除了将语音音频转换为文本，批量语音转文本还允许进行分割聚类和情感分析。 | 否 | [是](https://chinaeast2.cris.azure.cn/swagger/ui/index) |
| | [创建自定义语音识别模型](#customize-your-speech-experience) | 如果使用语音转文本在独特的环境中进行识别和听录，则可以创建并训练自定义的声学、语言和发音模型，以解决环境干扰或行业特定的词汇。 | 否 | [是](https://chinaeast2.cris.azure.cn/swagger/ui/index) |
| [文本转语音](text-to-speech.md) | 文本转语音 | 文本转语音可使用[语音合成标记语言 (SSML)](speech-synthesis-markup.md) 将输入文本转换为类似人类的合成语音。 可以选择标准语音或神经语音（请参阅[语言支持](language-support.md)）。 | [是](https://docs.azure.cn/cognitive-services/speech-service/speech-sdk-reference) | [是](https://docs.azure.cn/cognitive-services/speech-service/rest-apis) |
| [语音翻译](speech-translation.md) | 语音翻译 | 使用语音翻译可在应用程序、工具和设备中实现实时的多语言语音翻译。 进行语音转语音和语音转文本翻译时可以使用此服务。 | [是](https://docs.azure.cn/cognitive-services/speech-service/speech-sdk-reference) | 否 |


[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

## <a name="try-the-speech-service-for-free"></a>免费试用语音服务

若要完成以下步骤，需要一个 Azure 帐户。 如果你没有 Azure 帐户，可以在[此处](https://wd.azure.cn/pricing/1rmb-trial/)注册试用版。


### <a name="create-the-azure-resource"></a>创建 Azure 资源

若要将语音服务资源（试用层或付费层）添加到 Azure 帐户，请执行以下步骤：

1. 使用 Azure 帐户登录到 [Azure 门户](https://portal.azure.cn/)。

1. 选择门户左上角的“创建资源”。  如果未看到“创建资源”，可通过选择屏幕左上角的折叠菜单找到它。

1. 在“新建”窗口中的搜索框内键入“语音”，然后按 ENTER。 

1. 在搜索结果中，选择“语音”。 

   ![语音搜索结果](media/index/speech-search.png)

1. 选择“创建”，然后： 

   - 为新资源指定唯一的名称。 名称有助于区分绑定到同一服务的多个订阅。
   - 选择新资源关联的 Azure 订阅，以确定计费方式。
   - 选择将使用资源的[区域](regions.md)。
   - 选择免费 (F0) 或付费 (S0) 定价层。 若要查看每个层的定价和用量配额的完整信息，请选择“查看全部定价详细信息”  。
   - 为此“语音”订阅创建新的资源组或将订阅分配到现有资源组。 资源组有助于使多种 Azure 订阅保持有序状态。
   - 选择“创建”  。 系统随后会将你转到部署概述，并显示部署进度消息。
<!--
> [!NOTE]
> You can create an unlimited number of standard-tier subscriptions in one or multiple regions. However, you can create only one free-tier subscription. Model deployments on the free tier that remain unused for 7 days will be decommissioned automatically.
-->
部署新的语音资源需要花费片刻时间。 部署完成后，选择“转到资源”，然后在左侧导航窗格中选择“密钥”以显示语音服务订阅密钥。   每个订阅有两个密钥；可在应用程序中使用任意一个密钥。 若要将密钥快速复制/粘贴到代码编辑器或其他位置，请选择每个密钥旁边的复制按钮，切换窗口，然后将剪贴板中的内容粘贴到所需位置。

> [!IMPORTANT]
> 这些订阅密钥用于访问认知服务 API。 不要共享你的密钥。 安全存储密钥 - 例如，使用 Azure Key Vault。 此外，我们建议定期重新生成这些密钥。 发出 API 调用只需一个密钥。 重新生成第一个密钥时，可以使用第二个密钥来持续访问服务。

## <a name="complete-a-quickstart"></a>完成快速入门

我们提供了适用于大多数流行编程语言的快速入门，旨在让你了解基本设计模式并帮助你在 10 分钟以内运行代码。 请参阅以下列表，了解每项功能的快速入门。

* [语音转文本快速入门](get-started-speech-to-text.md)
* [文本转语音快速入门](get-started-text-to-speech.md)
* [语音翻译快速入门](speech-translation-basics.md)
* [意向识别快速入门](quickstarts/intent-recognition.md)

<!-- * [Speaker recognition quickstart](speaker-recognition-basics.md) -->

在你有机会开始使用语音服务后，请尝试一下我们的教程，了解如何处理各种情况。

- [教程：使用适用于 C# 的语音 SDK 和 LUIS 从语音中识别意向](how-to-recognize-intents-from-speech-csharp.md)

<!-- - [Tutorial: Voice enable your bot with the Speech SDK, C#](tutorial-voice-enable-your-bot-speech-sdk.md) -->
<!-- - [Tutorial: Build a Flask app to translate text, analyze sentiment, and synthesize translated text to speech, REST](https://docs.microsoft.com/azure/cognitive-services/translator/tutorial-build-flask-app-translation-synthesis?toc=%2fazure%2fcognitive-services%2fspeech-service%2ftoc.json&bc=%2fazure%2fcognitive-services%2fspeech-service%2fbreadcrumb%2ftoc.json&toc=%2Fen-us%2Fazure%2Fcognitive-services%2Fspeech-service%2Ftoc.json&bc=%2Fen-us%2Fazure%2Fbread%2Ftoc.json) -->

## <a name="get-sample-code"></a>获取示例代码

> [!IMPORTANT]
> 需要语音 SDK 版本 1.11.0 或更高版本。

GitHub 上提供了语音服务的示例代码。 这些示例涵盖了常见方案，例如，从文件或流中读取音频、连续和单次识别，以及使用自定义模型。 使用以下链接查看 SDK 和 REST 示例：

- [语音转文本、文本转语音和语音翻译示例 (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
- [批量听录示例 (REST)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/batch)
- [文本转语音示例 (REST)](https://github.com/Azure-Samples/Cognitive-Speech-TTS)

## <a name="customize-your-speech-experience"></a>自定义语音体验

语音服务能够很好地与内置模型配合工作，但是，你可能想要根据自己的产品或环境，进一步自定义和优化体验。 自定义选项的范围从声学模型优化，到专属于自有品牌的语音字体。

其他产品提供了针对特定用途（如卫生保健或保险）而优化的语音模型，但可供所有人平等地使用。 Azure 语音的自定义功能将成为你的独特竞争优势部分，而其他任何用户或客户都无法使用。 换句话说，你的模型是私人的，仅针对你的用例进行自定义调整。

| 语音服务 | 平台 | 说明 |
| -------------- | -------- | ----------- |
| 语音转文本 | [自定义语音识别](https://speech.azure.cn/customspeech) | 根据需要和可用数据自定义语音识别模型。 克服语音识别障碍，如说话风格、词汇和背景噪音。 |

## <a name="reference-docs"></a>参考文档

- [语音 SDK](speech-sdk-reference.md)
- [语音设备 SDK](speech-devices-sdk.md)
- [REST API：语音转文本](rest-speech-to-text.md)
- [REST API：文本转语音](rest-text-to-speech.md)
- [REST API：批量听录和自定义](https://chinaeast2.cris.azure.cn/swagger/ui/index)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [开始使用语音转文本](speech-to-text-basics.md)
> [开始使用文本转语音](get-started-text-to-speech.md)
