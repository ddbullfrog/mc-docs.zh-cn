---
title: 什么是 Azure 认知服务？
titleSuffix: Azure Cognitive Services
description: Azure 认知服务是包含 REST API 和客户端库 SDK 的云服务，可与 Azure 一起用于构建智能应用程序。
services: cognitive-services
author: Johnnytechn
manager: nitinme
keywords: cognitive services, cognitive intelligence, cognitive solutions, ai services, cognitive understanding, cognitive features
ms.service: cognitive-services
ms.subservice: ''
ms.topic: overview
ms.date: 10/28/2020
ms.author: v-johya
ms.custom: cog-serv-seo-aug-2020
ms.openlocfilehash: 0f00791fd0d9eeaca4968d68546b05564c6e75c8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106723"
---
# <a name="what-are-azure-cognitive-services"></a>什么是 Azure 认知服务？

Azure 认知服务是包含 REST API 和客户端库 SDK 的基于云的服务，可帮助开发人员将认知智能内置于应用程序，而无需具备直接的人工智能 (AI) 或数据科学技能或知识。 借助 Azure 认知服务，开发人员可以通过能够看、听、说、理解甚至开始推理的认知解决方案，轻松将认知功能添加到他们的应用程序中。

提供认知理解功能的 AI 服务主要分为五大类：

* 影像
* 语音
* 语言
* 决策

## <a name="vision-apis"></a>视觉 API

|服务名称|服务说明|
|:-----------|:------------------|
|[计算机视觉](/cognitive-services/computer-vision/ "计算机视觉")|使用计算机视觉服务，你可以访问用于处理图像并返回信息的高级认知算法。|
|[人脸](/cognitive-services/face/ "人脸")| 使用人脸服务可访问高级人脸算法，从而实现人脸属性检测和识别。|

## <a name="speech-apis"></a>语音 API

|服务名称|服务说明|
|:-----------|:------------------|
|[语音服务](/cognitive-services/speech-service/ "语音服务")|语音服务将支持语音的功能添加到应用程序。|

> [!NOTE]
> 正在查找 [Azure 认知搜索](/search/)？ 尽管它对某些任务使用认知服务，但它是一项支持其他方案的不同搜索技术。


## <a name="language-apis"></a>语言 API

|服务名称|服务说明|
|:-----------|:------------------|
|[语言理解 LUIS](/cognitive-services/luis/ "语言理解")|使用语言理解服务 (LUIS)，应用程序可以理解用户以自己的语言表达的内容。|
|[文本分析](/cognitive-services/text-analytics/ "文本分析")|文本分析提供对原始文本的自然语言处理，用于情绪分析、关键短语提取和语言检测。|
|[翻译](/cognitive-services/translator/ "转换器")|“翻译”近乎实时地提供基于机器的文本翻译。|


## <a name="decision-apis"></a>决策 API

|服务名称|服务说明|
|:-----------|:------------------|
|[内容审查器](/cognitive-services/content-moderator/overview "内容审查器")|内容审查器监视可能的冒犯性、不可取和危险内容。|

## <a name="learn-with-the-quickstarts"></a>通过快速入门学习

了解如何使用以下内容，通过实操性快速入门创建认知服务资源：

* [Azure 门户](cognitive-services-apis-create-account.md?tabs=multiservice%2Cwindows "Azure 门户")
* [Azure CLI](cognitive-services-apis-create-account-cli.md?tabs=windows "Azure CLI")
* [Azure SDK 客户端库](cognitive-services-apis-create-account-cli.md?tabs=windows "cognitive-services-apis-create-account-client-library?pivots=programming-language-csharp")

## <a name="upgrade-to-unlock-higher-limits"></a>升级以解锁更多限制

所有 API 都有一个免费层，它具有使用量和吞吐量限制。  在 Azure 门户中部署服务时，可以通过使用付费产品/服务和选择适当的定价层选项来增加这些限制。 [详细了解产品/服务和定价](https://www.azure.cn/pricing/details/cognitive-services/ "产品/服务和定价")。 你将需要使用信用卡和电话号码设置一个 Azure 订阅者帐户。 如果你有特殊要求或者只是想与销售人员交谈，请单击定价页顶部的“联系我们”按钮。

## <a name="regional-availability"></a>区域可用性

认知服务中的 API 托管在不断扩大的 Azure 托管数据中心网络上。 你可以在 [Azure 区域列表](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services "Azure 区域列表")中找到每个 API 的区域可用性。

## <a name="supported-cultural-languages"></a>支持的区域性语言

 认知服务在服务级别支持各种区域性语言。 可以在[支持的语言列表](language-support.md "支持的语言列表")中找到每个 API 的语言可用性。

## <a name="securing-resources"></a>保护资源

Azure 认知服务提供了分层的安全模型，包括通过 Azure Active Directory 凭据进行的[身份验证](authentication.md "身份验证")和有效的资源密钥。

## <a name="container-support"></a>容器支持

 认知服务提供用于在 Azure 云或本地部署的容器。 详细了解[认知服务容器](cognitive-services-container-support.md "认知服务容器")。

## <a name="certifications-and-compliance"></a>认证和合规性

认知服务已获得 CSA STAR 认证、FedRAMP 中等 和HIPAA BAA 等认证。

可以[下载](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942 "下载")认证进行自己的审核和安全评审。

若要了解隐私和数据管理，请访问[信任中心](https://servicetrust.microsoft.com/ "信任中心")。

## <a name="support"></a>支持

认知服务提供多个[支持选项](cognitive-services-support-options.md "支持选项")。




## <a name="next-steps"></a>后续步骤

* [创建认知服务帐户](cognitive-services-apis-create-account.md "创建认知服务帐户")

