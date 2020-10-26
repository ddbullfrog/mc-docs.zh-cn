---
title: 如何获取 V3 预测终结点
titleSuffix: Azure Cognitive Services
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: include
ms.date: 10/19/2020
ms.author: v-johya
ms.openlocfilehash: c4347089b91507406a906b20b00b9996b4eaa494
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472478"
---
1. 在 LUIS 门户中的“Azure 资源”页面（左菜单）上的“预测资源”选项卡的“管理”部分（右上菜单）中，复制页面底部的“示例查询”   。

    将 URL 粘贴到新的浏览器选项卡中。

    该 URL 具有应用 ID、密钥和槽名称。 V3 预测终结点 URL 如下所示：

    `https://YOUR-RESOURCE-NAME.api.cognitive.azure.cn/luis/prediction/v3.0/apps/APP-ID/slots/SLOT-NAME/predict?subscription-key=YOUR-PREDICTION-KEY&<optional-name-value-pairs>&query=YOUR_QUERY_HERE`


