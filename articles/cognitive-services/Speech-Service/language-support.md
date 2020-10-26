---
title: 语言支持 - 语音服务
titleSuffix: Azure Cognitive Services
description: 语音服务支持多种语言，可用于语音到文本和文本到语音转换，以及语音翻译。 本文提供了按服务功能列出的语言支持的完整列表。
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
origin.date: 03/26/2020
ms.date: 10/16/2020
ms.author: v-tawe
ms.custom: seodec18
ms.openlocfilehash: 668a229b866d3445de4848a170e726e4f839ae49
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128325"
---
# <a name="language-and-voice-support-for-the-speech-service"></a>语音服务的语言和语音支持

语言支持因语音服务功能而异。 下表汇总了对[语音转文本](#speech-to-text)、[文本转语音](#text-to-speech)和[语音翻译](#speech-translation)服务产品的语言支持。

## <a name="speech-to-text"></a>语音转文本

Microsoft 语音 SDK 和 REST API 都支持以下语言（区域设置）。 

为了提高准确性，已为一部分语言提供了自定义功能，你可通过上传音频和人工标记的脚本或相关文本（语句）进行自定义。 若要了解有关自定义的详细信息，请参阅[自定义语音识别入门](how-to-custom-speech.md)。

<!--
To get the AM and ML bits:
https://chinaeast2.dev.cognitive.azure.cn/docs/services/speech-to-text-api-v3-0/operations/GetSupportedLocalesForModels

To get pronunciation bits:
https://cris.azure.cn -> Click on Adaptation Data -> scroll down to section "Pronunciation Datasets" -> Click on Import -> Locale: the list of locales there correspond to the supported locales
-->

| Locale  | 语言                          | 支持 | 自定义                                    |
| ------- | --------------------------------- | --------- | ------------------------------------------------- |
| `ar-AE` | 阿拉伯语（阿拉伯联合酋长国）     | 是       | 语言模型                                    |
| `ar-BH` | 阿拉伯语(巴林)，现代标准 | 是       | 语言模型                                    |
| `ar-EG` | 阿拉伯语（埃及）                    | 是       | 语言模型                                    |
| `ar-IQ` | 阿拉伯语（伊拉克）                     | 是       | 语言模型                                    |
| `ar-JO` | 阿拉伯语（约旦）                   | 是       | 语言模型                                    |
| `ar-KW` | 阿拉伯语(科威特)                   | 是       | 语言模型                                    |
| `ar-LB` | 阿拉伯语（黎巴嫩）                  | 是       | 语言模型                                    |
| `ar-OM` | 阿拉伯语（阿曼）                     | 是       | 语言模型                                    |
| `ar-QA` | 阿拉伯语（卡塔尔）                    | 是       | 语言模型                                    |
| `ar-SA` | 阿拉伯语（沙特阿拉伯）             | 是       | 语言模型                                    |
| `ar-SY` | 阿拉伯语（叙利亚）                    | 是       | 语言模型                                    |
| `bg-BG` | 保加利亚语(保加利亚)              | 否        | 语言模型                                    |
| `ca-ES` | 加泰罗尼亚语(西班牙)                   | 否        | 语言模型                                    |
| `cs-CZ` | 捷克语（捷克共和国）            | 否        | 语言模型                                    |
| `da-DK` | 丹麦语（丹麦）                  | 否        | 语言模型                                    |
| `de-DE` | 德语（德国）                  | 是       | 声学模型<br>语言模型<br>发音 |
| `el-GR` | 希腊语(希腊)                    | 是        | 语言模型                                    |
| `en-AU` | 英语（澳大利亚）               | 是       | 声学模型<br>语言模型                  |
| `en-CA` | 英语（加拿大）                  | 是       | 声学模型<br>语言模型                  |
| `en-GB` | 英语（英国）          | 是       | 声学模型<br>语言模型<br>发音 |
| `en-HK` | 英语（香港）               | 是       | 语言模型                                    |
| `en-IE` | 英语（爱尔兰）                 | 是       | 语言模型                                    |
| `en-IN` | 英语（印度）                   | 是       | 声学模型<br>语言模型                  |
| `en-NZ` | 英语（新西兰）             | 是       | 声学模型<br>语言模型                  |
| `en-PH` | 英语（菲律宾）             | 是       | 语言模型                                    |
| `en-SG` | 英语（新加坡）               | 是       | 语言模型                                    |
| `en-US` | 英语（美国）           | 是       | 声学模型<br>语言模型<br>发音 |
| `en-ZA` | 英语（南非）            | 是       | 语言模型                                    |
| `es-AR` | 西班牙语（阿根廷）               | 是       | 语言模型                                    |
| `es-BO` | 西班牙语（玻利维亚）                 | 是       | 语言模型                                    |
| `es-CL` | 西班牙语（智利）                   | 是       | 语言模型                                    |
| `es-CO` | 西班牙语（哥伦比亚）                | 是       | 语言模型                                    |
| `es-CR` | 西班牙语（哥斯达黎加）              | 是       | 语言模型                                    |
| `es-CU` | 西班牙语（古巴）                    | 是       | 语言模型                                    |
| `es-DO` | 西班牙语（多米尼加共和国）      | 是       | 语言模型                                    |
| `es-EC` | 西班牙语（厄瓜多尔）                 | 是       | 语言模型                                    |
| `es-ES` | 西班牙语(西班牙)                   | 是       | 声学模型<br>语言模型                  |
| `es-GT` | 西班牙语（危地马拉）               | 是       | 语言模型                                    |
| `es-HN` | 西班牙语（洪都拉斯）                | 是       | 语言模型                                    |
| `es-MX` | 西班牙语（墨西哥）                  | 是       | 声学模型<br>语言模型                  |
| `es-NI` | 西班牙（尼加拉瓜）               | 是       | 语言模型                                    |
| `es-PA` | 西班牙语（巴拿马）                  | 是       | 语言模型                                    |
| `es-PE` | 西班牙语（秘鲁）                    | 是       | 语言模型                                    |
| `es-PR` | 西班牙语（波多黎各）             | 是       | 语言模型                                    |
| `es-PY` | 西班牙语（巴拉圭）                | 是       | 语言模型                                    |
| `es-SV` | 西班牙语（萨尔瓦多）             | 是       | 语言模型                                    |
| `es-US` | 西班牙语（美国）                     | 是       | 语言模型                                    |
| `es-UY` | 西班牙语（乌拉圭）                 | 是       | 语言模型                                    |
| `es-VE` | 西班牙语（委内瑞拉）               | 是       | 语言模型                                    |
| `et-EE` | 爱沙尼亚语（爱沙尼亚）                 | 否        | 语言模型                                    |
| `fi-FI` | 芬兰语（芬兰）                 | 否        | 语言模型                                    |
| `fr-CA` | 法语（加拿大）                   | 是       | 声学模型<br>语言模型                  |
| `fr-FR` | 法语（法国）                   | 是       | 声学模型<br>语言模型<br>发音 |
| `ga-IE` | 爱尔兰语（爱尔兰）                    | 否        | 语言模型                                    |
| `gu-IN` | 古吉拉特语(印度)                 | 否        | 语言模型                                    |
| `hi-IN` | 印地语（印度）                     | 是       | 声学模型<br>语言模型                  |
| `hr-HR` | 克罗地亚语（克罗地亚）                | 否        | 语言模型                                    |
| `hu-HU` | 匈牙利语（匈牙利）               | 是       | 语言模型                                    |
| `it-IT` | 意大利语（意大利）                   | 否        | 声学模型<br>语言模型<br>发音 |
| `ja-JP` | 日语（日本）                  | 否        | 语言模型                                    |
| `ko-KR` | 韩语(韩国)                    | 是       | 语言模型                                    |
| `lt-LT` | 立陶宛语（立陶宛）            | 否        | 语言模型                                    |
| `lv-LV` | 拉脱维亚语（拉脱维亚）                  | 否        | 语言模型                                    |
| `mr-IN` | 马拉地语(印度)                   | 否        | 语言模型                                    |
| `mt-MT` | 马耳他语（马耳他）                    | 否        | 语言模型                                    |
| `nb-NO` | 书面挪威语(挪威)       | 否        | 语言模型                                    |
| `nl-NL` | 荷兰语（荷兰）               | 否        | 语言模型                                    |
| `pl-PL` | 波兰语（波兰）                   | 否        | 语言模型                                    |
| `pt-BR` | 葡萄牙语（巴西）               | 否        | 声学模型<br>语言模型<br>发音 |
| `pt-PT` | 葡萄牙语(葡萄牙)             | 否        | 语言模型                                    |
| `ro-RO` | 罗马尼亚语（罗马尼亚）                | 否        | 语言模型                                    |
| `ru-RU` | 俄语（俄罗斯）                  | 是       | 声学模型<br>语言模型                  |
| `sk-SK` | 斯洛伐克语（斯洛伐克）                 | 否        | 语言模型                                    |
| `sl-SI` | 斯洛文尼亚语（斯洛文尼亚）              | 否        | 语言模型                                    |
| `sv-SE` | 瑞典语（瑞典）                  | 否        | 语言模型                                    |
| `ta-IN` | 泰米尔语（印度）                     | 否        | 语言模型                                    |
| `te-IN` | 泰卢固语（印度）                    | 否        | 语言模型                                    |
| `th-TH` | 泰语（泰国）                   | 否        | 语言模型                                    |
| `tr-TR` | 土耳其语（土耳其）                  | 否        | 语言模型                                    |
| `zh-CN` | 中文（普通话，简体）    | 是       | 声学模型<br>语言模型                  |
| `zh-HK` | 中文(粤语，繁体)  | 是       | 语言模型                                    |
| `zh-TW` | 中文(台湾普通话)      | 是       | 语言模型                                    |

## <a name="text-to-speech"></a>文本转语音

Microsoft 语音 SDK 和 REST API 支持以下语音，其中的每种语音都支持特定语言和方言（按区域设置标识）。 还可以通过[语音/列表 API](rest-text-to-speech.md#get-a-list-of-voices) 获取每个特定区域/终结点支持的语言和语音的完整列表。 

> [!IMPORTANT]
> 标准语音和神经语音的定价各不相同。 有关其他信息，请访问[定价](https://www.azure.cn/pricing/details/cognitive-services/)页。

### <a name="neural-voices"></a>神经语音

神经文本到语音转换是由深度神经网络提供支持的新型语音合成。 使用神经语音时，几乎无法将合成的语音与人类录音区分开来。

使用神经语音可使得与聊天机器人和语音助手的交互更加自然且富有吸引力、将数字文本（如电子书）转换为有声读物以及增强车载导航系统。 随着类人的自然韵律和字词的清晰发音，用户在与 AI 系统交互时，神经语音显著减轻了听力疲劳。

有关区域可用性的详细信息，请参阅[区域](regions.md#standard-and-neural-voices)。

| Locale                 | 语言                                   | 支持 | 性别 | 语音名称              | 风格支持                                               |
| ---------------------- | ------------------------------------------ | --------- | ------ | ----------------------- | ----------------------------------------------------------- |
| `ar-EG`                | 阿拉伯语（埃及）                             | 否        | 女 | `ar-EG-SalmaNeural`     | 常规                                                     |
| `ar-SA`                | 阿拉伯语（沙特阿拉伯）                      | 否        | 女 | `ar-SA-ZariyahNeural`   | 常规                                                     |
| `bg-BG` <sup>新建</sup> | 保加利亚语（保加利亚）                        | 否        | 女 | `bg-BG-KalinaNeural`    | 常规                                                     |
| `ca-ES`                | 加泰罗尼亚语(西班牙)                            | 否        | 女 | `ca-ES-AlbaNeural`      | 常规                                                     |
| `cs-CZ` <sup>新建</sup> | 捷克语（捷克）                              | 否        | 女 | `cs-CZ-VlastaNeural`    | 常规                                                     |
| `da-DK`                | 丹麦语（丹麦）                           | 否        | 女 | `da-DK-ChristelNeural`  | 常规                                                     |
| `de-AT` <sup>新建</sup> | 德语（奥地利）                           | 否        | 女 | `de-AT-IngridNeural`    | 常规                                                     |
| `de-CH` <sup>新建</sup> | 德语（瑞士）                       | 否        | 女 | `de-CH-LeniNeural`      | 常规                                                     |
| `de-DE`                | 德语（德国）                           | 是       | 女 | `de-DE-KatjaNeural`     | 常规                                                     |
| `de-DE` <sup>新建</sup> | 德语（德国）                           | 否        | 男   | `de-DE-ConradNeural`    | 常规                                                     |
| `el-GR` <sup>新建</sup> | 希腊语(希腊)                             | 是       | 女 | `el-GR-AthinaNeural`    | 常规                                                     |
| `en-AU`                | 英语（澳大利亚）                        | 是       | 女 | `en-AU-NatashaNeural`   | 常规                                                     |
| `en-AU` <sup>新建</sup> | 澳大利亚英语（澳大利亚）                     | 是       | 男   | `en-AU-WilliamNeural`   | 常规                                                     |
| `en-CA`                | 英语（加拿大）                           | 是       | 女 | `en-CA-ClaraNeural`     | 常规                                                     |
| `en-GB`                | 英语（英国）                  | 是       | 女 | `en-GB-LibbyNeural`     | 常规                                                     |
| `en-GB`                | 英语（英国）                  | 是       | 女 | `en-GB-MiaNeural`       | 常规                                                     |
| `en-GB` <sup>新建</sup> | 英语（英国）                   | 是       | 男   | `En-GB-RyanNeural`      | 常规                                                     |
| `en-IE` <sup>新建</sup> | 爱尔兰英语（爱尔兰）                    | 是       | 女 | `en-IE-EmilyNeural`     | 常规                                                     |
| `en-IN`                | 英语（印度）                            | 是       | 女 | `en-IN-NeerjaNeural`    | 常规                                                     |
| `en-US`                | 英语（美国）                   | 是       | 女 | `en-US-AriaNeural`      | 常规，提供了多种语音风格                    |
| `en-US`                | 英语（美国）                   | 是       | 男   | `en-US-GuyNeural`       | 常规                                                     |
| `en-US` <sup>新建</sup> | 英语（美国）                   | 是       | 女 | `en-US-JennyNeural`     | 常规，提供了多种语音风格                    |
| `es-ES`                | 西班牙语(西班牙)                            | 否        | 女 | `es-ES-ElviraNeural`    | 常规                                                     |
| `es-ES` <sup>新建</sup> | 西班牙语（西班牙）                           | 否        | 男   | `es-ES-AlvaroNeural`    | 常规                                                     |
| `es-MX`                | 西班牙语（墨西哥）                           | 否        | 女 | `es-MX-DaliaNeural`     | 常规                                                     |
| `es-MX` <sup>新建</sup> | 西班牙语（墨西哥）                           | 否        | 男   | `es-MX-JorgeNeural`     | 常规                                                     |
| `fi-FI`                | 芬兰语（芬兰）                          | 否        | 女 | `fi-FI-NooraNeural`     | 常规                                                     |
| `fr-CA`                | 法语（加拿大）                            | 否        | 女 | `fr-CA-SylvieNeural`    | 常规                                                     |
| `fr-CA` <sup>新建</sup> | 法语（加拿大）                            | 否        | 男   | `fr-CA-JeanNeural`      | 常规                                                     |
| `fr-CH` <sup>新建</sup> | 法语（瑞士）                       | 否        | 女 | `fr-CH-ArianeNeural`    | 常规                                                     |
| `fr-FR`                | 法语（法国）                            | 否        | 女 | `fr-FR-DeniseNeural`    | 常规                                                     |
| `fr-FR` <sup>新建</sup> | 法语（法国）                            | 否        | 男   | `fr-FR-HenriNeural`     | 常规                                                     |
| `he-IL` <sup>新建</sup> | 希伯来语（以色列）                           | 否        | 女 | `he-IL-HilaNeural`      | 常规                                                     |
| `hi-IN`                | 印地语（印度）                              | 否        | 女 | `hi-IN-SwaraNeural`     | 常规                                                     |
| `hr-HR` <sup>新建</sup> | 克罗地亚语（克罗地亚）                         | 否        | 女 | `hr-HR-GabrijelaNeural` | 常规                                                     |
| `hu-HU` <sup>新建</sup> | 匈牙利语(匈牙利)                        | 否        | 女 | `hu-HU-NoemiNeural`     | 常规                                                     |
| `id-ID` <sup>新建</sup> | 印度尼西亚语（印度尼西亚）              | 否        | 男   | `id-ID-ArdiNeural`      | 常规                                                     |
| `it-IT`                | 意大利语（意大利）                            | 否        | 女 | `it-IT-ElsaNeural`      | 常规                                                     |
| `it-IT` <sup>新建</sup> | 意大利语（意大利）                            | 否        | 女 | `it-IT-IsabellaNeural`  | 常规                                                     |
| `it-IT` <sup>新建</sup> | 意大利语（意大利）                            | 否        | 男   | `it-IT-DiegoNeural`     | 常规                                                     |
| `ja-JP`                | 日语（日本）                           | 否        | 女 | `ja-JP-NanamiNeural`    | 常规                                                     |
| `ja-JP` <sup>新建</sup> | 日语（日本）                           | 否        | 男   | `ja-JP-KeitaNeural`     | 常规                                                     |
| `ko-KR`                | 韩语(韩国)                             | 否        | 女 | `ko-KR-SunHiNeural`     | 常规                                                     |
| `ko-KR` <sup>新建</sup> | 韩语(韩国)                             | 否        | 男   | `ko-KR-InJoonNeural`    | 常规                                                     |
| `ms-MY` <sup>新建</sup> | 马来语（马来西亚）                           | 否        | 女 | `ms-MY-YasminNeural`    | 常规                                                     |
| `nb-NO`                | 挪威语，博克马尔语（挪威）                | 否        | 女 | `nb-NO-IselinNeural`    | 常规                                                     |
| `nl-NL`                | 荷兰语（荷兰）                        | 否        | 女 | `nl-NL-ColetteNeural`   | 常规                                                     |
| `pl-PL`                | 波兰语（波兰）                            | 否        | 女 | `pl-PL-ZofiaNeural`     | 常规                                                     |
| `pt-BR`                | 葡萄牙语（巴西）                        | 否        | 女 | `pt-BR-FranciscaNeural` | 常规，提供了多种语音风格                    |
| `pt-BR` <sup>新建</sup> | 巴西葡萄牙语（巴西）              | 否        | 男   | `pt-BR-AntonioNeural`   | 常规                                                     |
| `pt-PT`                | 葡萄牙语（葡萄牙）                     | 否        | 女 | `pt-PT-FernandaNeural`  | 常规                                                     |
| `ro-RO` <sup>新建</sup> | 罗马尼亚语（罗马尼亚）                         | 否        | 女 | `ro-RO-AlinaNeural`     | 常规                                                     |
| `ru-RU`                | 俄语（俄罗斯）                           | 否        | 女 | `ru-RU-DariyaNeural`    | 常规                                                     |
| `sk-SK` <sup>新建</sup> | 斯洛伐克语（斯洛伐克）                          | 否        | 女 | `sk-SK-ViktoriaNeural`  | 常规                                                     |
| `sl-SI` <sup>新建</sup> | 斯洛文尼亚语（斯洛文尼亚）                       | 否        | 女 | `sl-SI-PetraNeural`     | 常规                                                     |
| `sv-SE`                | 瑞典语（瑞典）                           | 否        | 女 | `sv-SE-HilleviNeural`   | 常规                                                     |
| `ta-IN` <sup>新建</sup> | 泰米尔语（印度）                              | 否        | 女 | `ta-IN-PallaviNeural`   | 常规                                                     |
| `te-IN` <sup>新建</sup> | 泰卢固语（印度）                             | 否        | 女 | `te-IN-ShrutiNeural`    | 常规                                                     |
| `th-TH`                | 泰语（泰国）                            | 否        | 女 | `th-TH-AcharaNeural`    | 常规                                                     |
| `th-TH` <sup>新建</sup> | 泰语（泰国）                            | 否        | 女 | `th-TH-PremwadeeNeural` | 常规                                                     |
| `tr-TR`                | 土耳其语（土耳其）                          | 否        | 女 | `tr-TR-EmelNeural`      | 常规                                                     |
| `vi-VN` <sup>新建</sup> | 越南语(越南)                       | 否        | 女 | `vi-VN-HoaiMyNeural`    | 常规                                                     |
| `zh-CN`                | 普通话（简体中文，中国）       | 否        | 女 | `zh-CN-XiaoxiaoNeural`  | 常规，提供了多种语音风格                    |
| `zh-CN`                | 普通话（简体中文，中国）       | 否        | 女 | `zh-CN-XiaoyouNeural`   | 儿童语音，针对讲故事进行了优化                    |
| `zh-CN`                | 普通话（简体中文，中国）       | 否        | 男   | `zh-CN-YunyangNeural`   | 针对新闻播报进行了优化，提供了多种语音风格 |
| `zh-CN`                | 普通话（简体中文，中国）       | 否        | 男   | `zh-CN-YunyeNeural`     | 针对讲故事进行了优化                               |
| `zh-HK`                | 粤语（繁体中文，香港） | 否        | 女 | `zh-HK-HiuGaaiNeural`   | 常规                                                     |
| `zh-TW`                | 普通话（繁体中文，台湾）     | 否        | 女 | `zh-TW-HsiaoYuNeural`   | 常规                                                     |

> [!IMPORTANT]
> `en-US-JessaNeural` 语音已更改为 `en-US-AriaNeural`。 如果以前使用了“Jessa”，请转换为“Aria”。

若要了解如何配置和调整神经语音，请参阅[语音合成标记语言](speech-synthesis-markup.md#adjust-speaking-styles)。

> [!TIP]
> 可以继续在语音合成请求中使用完整的服务名称映射，如“Microsoft Server Speech Text to Speech Voice (en-US, AriaNeural)”。

### <a name="standard-voices"></a>标准语音

40 多种标准语音在 10 多种语言和区域设置中提供，允许你将文本转换为合成语音。 有关区域可用性的详细信息，请参阅[区域](regions.md#standard-and-neural-voices)。

| Locale  | 语言                                   | 支持 | 性别 | 语音名称           |
| ------- | ------------------------------------------ | --------- | ------ | -------------------- |
| `ar-EG` | 阿拉伯语（阿拉伯）                           | 是       | Female | `ar-EG-Hoda`         |
| `ar-SA` | 阿拉伯语（沙特阿拉伯）                      | 是       | 男   | `ar-SA-Naayf`        |
| `bg-BG` | 保加利亚语(保加利亚)                       | 否        | 男   | `bg-BG-Ivan`         |
| `ca-ES` | 加泰罗尼亚语(西班牙)                            | 否        | 女 | `ca-ES-HerenaRUS`    |
| `cs-CZ` | 捷克语（捷克共和国）                     | 否        | 男   | `cs-CZ-Jakub`        |
| `da-DK` | 丹麦语（丹麦）                           | 否        | 女 | `da-DK-HelleRUS`     |
| `de-AT` | 德语（奥地利）                           | 是       | 男   | `de-AT-Michael`      |
| `de-CH` | 德语（瑞士）                       | 是       | 男   | `de-CH-Karsten`      |
| `de-DE` | 德语（德国）                           | 是       | Female | `de-DE-HeddaRUS`     |
| `de-DE` | 德语（德国）                           | 是       | 男   | `de-DE-Stefan`       |
| `el-GR` | 希腊语(希腊)                             | 否        | 男   | `el-GR-Stefanos`     |
| `en-AU` | 英语（澳大利亚）                        | 是       | Female | `en-AU-Catherine`    |
| `en-AU` | 英语（澳大利亚）                        | 是       | Female | `en-AU-HayleyRUS`    |
| `en-CA` | 英语（加拿大）                           | 是       | Female | `en-CA-HeatherRUS`   |
| `en-CA` | 英语（加拿大）                           | 是       | Female | `en-CA-Linda`        |
| `en-GB` | 英语（英国）                   | 是       | 男   | `en-GB-George`       |
| `en-GB` | 英语（英国）                   | 是       | Female | `en-GB-HazelRUS`     |
| `en-GB` | 英语（英国）                   | 是       | Female | `en-GB-Susan`        |
| `en-IE` | 英语（爱尔兰）                          | 是       | 男   | `en-IE-Sean`         |
| `en-IN` | 英语（印度）                            | 是       | Female | `en-IN-Heera`        |
| `en-IN` | 英语（印度）                            | 是       | Female | `en-IN-PriyaRUS`     |
| `en-IN` | 英语（印度）                            | 是       | 男   | `en-IN-Ravi`         |
| `en-US` | 英语（美国）                    | 是       | 男   | `en-US-BenjaminRUS`  |
| `en-US` | 英语（美国）                    | 是       | 男   | `en-US-GuyRUS`       |
| `en-US` | 英语（美国）                    | 是       | Female | `en-US-JessaRUS`     |
| `en-US` | 英语（美国）                    | 是       | Female | `en-US-ZiraRUS`      |
| `es-ES` | 西班牙语(西班牙)                            | 是       | Female | `es-ES-HelenaRUS`    |
| `es-ES` | 西班牙语(西班牙)                            | 是       | Female | `es-ES-Laura`        |
| `es-ES` | 西班牙语(西班牙)                            | 是       | 男   | `es-ES-Pablo`        |
| `es-MX` | 西班牙语（墨西哥）                           | 是       | Female | `es-MX-HildaRUS`     |
| `es-MX` | 西班牙语（墨西哥）                           | 是       | 男   | `es-MX-Raul`         |
| `fi-FI` | 芬兰语（芬兰）                          | 否        | 女 | `fi-FI-HeidiRUS`     |
| `fr-CA` | 法语（加拿大）                            | 是       | Female | `fr-CA-Caroline`     |
| `fr-CA` | 法语（加拿大）                            | 是       | Female | `fr-CA-HarmonieRUS`  |
| `fr-CH` | 法语（瑞士）                       | 是       | 男   | `fr-CH-Guillaume`    |
| `fr-FR` | 法语（法国）                            | 是       | Female | `fr-FR-HortenseRUS`  |
| `fr-FR` | 法语（法国）                            | 是       | Female | `fr-FR-Julie`        |
| `fr-FR` | 法语（法国）                            | 是       | 男   | `fr-FR-Paul`         |
| `he-IL` | 希伯来语（以色列）                            | 否        | 男   | `he-IL-Asaf`         |
| `hi-IN` | 印地语（印度）                              | 是       | 男   | `hi-IN-Hemant`       |
| `hi-IN` | 印地语（印度）                              | 是       | Female | `hi-IN-Kalpana`      |
| `hr-HR` | 克罗地亚语（克罗地亚）                         | 否        | 男   | `hr-HR-Matej`        |
| `hu-HU` | 匈牙利语(匈牙利)                        | 否        | 男   | `hu-HU-Szabolcs`     |
| `id-ID` | 印度尼西亚语(印度尼西亚)                     | 否        | 男   | `id-ID-Andika`       |
| `it-IT` | 意大利语（意大利）                            | 是       | 男   | `it-IT-Cosimo`       |
| `it-IT` | 意大利语（意大利）                            | 是       | Female | `it-IT-LuciaRUS`     |
| `ja-JP` | 日语（日本）                           | 否        | 女 | `ja-JP-Ayumi`        |
| `ja-JP` | 日语（日本）                           | 否        | 女 | `ja-JP-HarukaRUS`    |
| `ja-JP` | 日语（日本）                           | 否        | 男   | `ja-JP-Ichiro`       |
| `ko-KR` | 韩语(韩国)                             | 是       | Female | `ko-KR-HeamiRUS`     |
| `ms-MY` | 马来语（马来西亚）                           | 否        | 男   | `ms-MY-Rizwan`       |
| `nb-NO` | 挪威语(博克马尔语，挪威)                 | 否        | 女 | `nb-NO-HuldaRUS`     |
| `nl-NL` | 荷兰语（荷兰）                        | 否        | 女 | `nl-NL-HannaRUS`     |
| `pl-PL` | 波兰语（波兰）                            | 否        | 女 | `pl-PL-PaulinaRUS`   |
| `pt-BR` | 葡萄牙语（巴西）                        | 否        | 男   | `pt-BR-Daniel`       |
| `pt-BR` | 葡萄牙语（巴西）                        | 否        | 女 | `pt-BR-HeloisaRUS`   |
| `pt-PT` | 葡萄牙语(葡萄牙)                      | 否        | 女 | `pt-PT-HeliaRUS`     |
| `ro-RO` | 罗马尼亚语（罗马尼亚）                         | 否        | 男   | `ro-RO-Andrei`       |
| `ru-RU` | 俄语（俄罗斯）                           | 是       | Female | `ru-RU-EkaterinaRUS` |
| `ru-RU` | 俄语（俄罗斯）                           | 是       | Female | `ru-RU-Irina`        |
| `ru-RU` | 俄语（俄罗斯）                           | 是       | 男   | `ru-RU-Pavel`        |
| `sk-SK` | 斯洛伐克语（斯洛伐克）                          | 否        | 男   | `sk-SK-Filip`        |
| `sl-SI` | 斯洛文尼亚语（斯洛文尼亚）                       | 否        | 男   | `sl-SI-Lado`         |
| `sv-SE` | 瑞典语（瑞典）                           | 否        | 女 | `sv-SE-HedvigRUS`    |
| `ta-IN` | 泰米尔语（印度）                              | 否        | 男   | `ta-IN-Valluvar`     |
| `te-IN` | 泰卢固语（印度）                             | 否        | 女 | `te-IN-Chitra`       |
| `th-TH` | 泰语（泰国）                            | 否        | 男   | `th-TH-Pattara`      |
| `tr-TR` | 土耳其语（土耳其）                           | 否        | 女 | `tr-TR-SedaRUS`      |
| `vi-VN` | 越南语(越南)                       | 否        | 男   | `vi-VN-An`           |
| `zh-CN` | 普通话（简体中文，中国）       | 是       | Female | `zh-CN-HuihuiRUS`    |
| `zh-CN` | 普通话（简体中文，中国）       | 是       | 男   | `zh-CN-Kangkang`     |
| `zh-CN` | 普通话（简体中文，中国）       | 是       | Female | `zh-CN-Yaoyao`       |
| `zh-HK` | 粤语（繁体中文，香港） | 是       | 男   | `zh-HK-Danny`        |
| `zh-HK` | 粤语（繁体中文，香港） | 是       | Female | `zh-HK-TracyRUS`     |
| `zh-TW` | 普通话（繁体中文，台湾）     | 是       | Female | `zh-TW-HanHanRUS`    |
| `zh-TW` | 普通话（繁体中文，台湾）     | 是       | Female | `zh-TW-Yating`       |
| `zh-TW` | 普通话（繁体中文，台湾）     | 是       | 男   | `zh-TW-Zhiwei`       |

**1** ar-EG 支持现代标准阿拉伯语 (MSA)。

> [!IMPORTANT]
> `en-US-Jessa` 语音已更改为 `en-US-Aria`。 如果以前使用了“Jessa”，请转换为“Aria”。

> [!TIP]
> 可以继续在语音合成请求中使用完整的服务名称映射，如“Microsoft Server Speech Text to Speech Voice (en-US, AriaRUS)”。

### <a name="customization"></a>自定义

语音自定义适用于 `en-GB`、`en-IN`、`en-US`、`es-MX` 和 `zh-CN`。 选择与训练自定义语音模型所需的训练数据相匹配的正确区域设置。 例如，如果你的录音数据是以带英国口音的英语说出的，请选择 `en-GB`。

> [!NOTE]
> 除了中英双语模型之外，我们在自定义语音中不支持其他双语模型训练。 如果要训练一种也可以说英语的中文语音，请选择“中英双语”。 对于 `en-US` 和 `zh-CN` 之外的所有区域设置，语音训练都从一个包含 2000 条以上言语的数据集开始。对于这例外的两种区域设置，你可以从任何大小的训练数据开始。

## <a name="speech-translation"></a>语音翻译

**语音翻译** API 支持使用不同的语言进行语音转语音和语音转文本的翻译。 源语言必须始终来自“语音转文本”语言表。 可用的目标语言取决于翻译目标是语音还是文本。 可以将传入的语音翻译成 [60 种以上的语言](https://www.microsoft.com/translator/business/languages/)。 这些语言的子集可用于[语音合成](language-support.md#text-languages)。

### <a name="text-languages"></a>文本语言

| 文本语言           | 语言代码 | 支持 |
| :---------------------- | :-----------: | :-------: |
| 南非荷兰语               |     `af`      |    否     |
| 阿拉伯语                  |     `ar`      |    是    |
| Bangla                  |     `bn`      |    否     |
| 波斯尼亚语(拉丁语系)         |     `bs`      |    否     |
| 保加利亚语               |     `bg`      |    否     |
| 粤语(繁体) |     `yue`     |    否     |
| 加泰罗尼亚语                 |     `ca`      |    否     |
| 简体中文      |   `zh-Hans`   |    是    |
| 中文(繁体)     |   `zh-Hant`   |    是    |
| 克罗地亚语                |     `hr`      |    否     |
| 捷克语                   |     `cs`      |    否     |
| 丹麦语                  |     `da`      |    否     |
| 荷兰语                   |     `nl`      |    否     |
| 英语                 |     `en`      |    是    |
| 爱沙尼亚语                |     `et`      |    否     |
| 斐济语                  |     `fj`      |    否     |
| 菲律宾语                |     `fil`     |    否     |
| 芬兰语                 |     `fi`      |    否     |
| 法语                  |     `fr`      |    是    |
| 德语                  |     `de`      |    是    |
| 希腊语                   |     `el`      |    否     |
| 古吉拉特语                |     `gu`      |    否     |
| 海地克里奥尔语          |     `ht`      |    否     |
| 希伯来语                  |     `he`      |    否     |
| Hindi                   |     `hi`      |    是    |
| 白苗文               |     `mww`     |    否     |
| 匈牙利语               |     `hu`      |    否     |
| 印度尼西亚语              |     `id`      |    否     |
| 爱尔兰语                   |     `ga`      |    否     |
| 意大利语                 |     `it`      |    否     |
| 日语                |     `ja`      |    否     |
| 卡纳达语                 |     `kn`      |    否     |
| 斯瓦希里语               |     `sw`      |    否     |
| 克林贡语                 |  `tlh-Latn`   |    否     |
| 克林贡语(plqaD)         |  `tlh-Piqd`   |    否     |
| 朝鲜语                  |     `ko`      |    是    |
| 拉脱维亚语                 |     `lv`      |    否     |
| 立陶宛语              |     `lt`      |    否     |
| 马达加斯加语                |     `mg`      |    否     |
| 马来语                   |     `ms`      |    否     |
| 马拉雅拉姆语               |     `ml`      |    否     |
| 马耳他语                 |     `mt`      |    否     |
| 毛利语                   |     `mi`      |    否     |
| 马拉地语                 |     `mr`      |    否     |
| 挪威语               |     `nb`      |    否     |
| 波斯语                 |     `fa`      |    否     |
| 波兰语                  |     `pl`      |    否     |
| 葡萄牙语（巴西）     |    `pt-br`    |    否     |
| 葡萄牙语(葡萄牙)   |    `pt-pt`    |    否     |
| 旁遮普语                 |     `pa`      |    否     |
| 克雷塔罗奥托米语         |     `otq`     |    否     |
| 罗马尼亚语                |     `ro`      |    否     |
| 俄语                 |     `ru`      |    是    |
| 萨摩亚语                  |     `sm`      |    否     |
| 塞尔维亚语（西里尔）      |   `sr-Cyrl`   |    否     |
| 塞尔维亚语（拉丁）         |   `sr-Latn`   |    否     |
| 斯洛伐克语                  |     `sk`      |    否     |
| 斯洛文尼亚语               |     `sl`      |    否     |
| 西班牙语                 |     `es`      |    是    |
| 瑞典语                 |     `sv`      |    否     |
| 塔希提语                |     `ty`      |    否     |
| 泰米尔语                   |     `ta`      |    否     |
| 泰卢固语                  |     `te`      |    否     |
| 泰语                    |     `th`      |    否     |
| 汤加语                  |     `to`      |    否     |
| 土耳其语                 |     `tr`      |    否     |
| 乌克兰语               |     `uk`      |    否     |
| 乌尔都语                    |     `ur`      |    否     |
| 越南语              |     `vi`      |    否     |
| 威尔士语                   |     `cy`      |    否     |
| 尤卡坦玛雅语            |     `yua`     |    否     |

<!-- ## Speaker Recognition -->

## <a name="more-languages"></a>更多语言

> [!NOTE]
> 如果前面的列表中不支持你指定的语言，请参考[主权云支持的区域设置](sovereign-clouds.md)和[联系支持](https://www.azure.cn/support/contact/)，可以根据要求部署该语言。


## <a name="next-steps"></a>后续步骤

* [创建一个 Azure 试用帐户](https://www.azure.cn/home/features/cognitive-services/)
* [了解如何在 C# 中识别语音](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-chsarp)
