---
title: 发布区域和终结点 - LUIS
description: Azure 门户中指定的区域就是你将在其中发布 LUIS 应用的区域，并会为此同一区域生成一个终结点 URL。
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 11/19/2019
ms.author: v-johya
ms.openlocfilehash: 755f63e92dd08aaee18b6ff5d2c972fc0c81c1aa
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472360"
---
# <a name="authoring-and-publishing-regions-and-the-associated-keys"></a>创作和发布区域及关联的密钥

若要将 LUIS 应用发布到多个区域，每个区域至少需要一个密钥。

<a name="luis-website"></a>

## <a name="luis-authoring-regions"></a>LUIS 创作区域
必须在同一区域中创建和发布应用。

|LUIS|创作区域|Azure 区域名称|
|--|--|--|
|[luis.azure.cn][luis.azure.cn]|中国| `chinaeast2`|

创作区域具有[配对故障转移区域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)。

<a name="regions-and-azure-resources"></a>

## <a name="publishing-regions-and-azure-resources"></a>发布区域和 Azure 资源 
该应用将发布到与 LUIS 门户中添加的 LUIS 资源关联的所有区域。 例如，对于在 [luis.azure.cn][luis.azure.cn] 上创建的应用，如果你在 chinaeast2 中创建 LUIS 或认知服务资源并[将其作为资源添加到该应用](luis-how-to-azure-subscription.md)，则该应用将发布到此区域中。 

## <a name="public-apps"></a>公共应用
公共应用在所有区域中发布，以便有基于区域的 LUIS 资源密钥的用户可以在与其资源密钥关联的任何区域中访问该应用。

<a name="publishing-regions"></a>

## <a name="publishing-regions-are-tied-to-authoring-regions"></a>发布区域绑定到创作区域

创作区域中的应用仅可发布到对应的发布区域。 如果应用目前位于错误的创作区域中，请导出应用，然后将其导入发布区域对应的正确创作区域。

 全球区域 | 创作 API 区域和创作网站| 发布和查询区域<br>`API region name`   |  终结点 URL 格式   |
|-----|------|------|------|
| 中国 | `chinaeast2`<br>[luis.azure.cn][luis.azure.cn]| 中国东部 2<br>`chinaeast2`     |  https://chinaeast2.api.cognitive.azure.cn/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY   |

## <a name="endpoints"></a>终结点

详细了解[创作和预测终结点](developer-reference-resource.md)。

## <a name="failover-regions"></a>故障转移区域

每个区域都有一个要故障转移到的次要区域。

创作区域具有[配对故障转移区域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [预生成实体参考](./luis-reference-prebuilt-entities.md)

 [luis.azure.cn]: https://luis.azure.cn
