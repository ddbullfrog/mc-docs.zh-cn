---
author: Johnnytechn
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: include
ms.date: 10/22/2020
ms.author: v-johya
ms.openlocfilehash: dc347b29c905fef816b63ea08bdfe20f5d717e11
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92499210"
---
## <a name="set-up"></a>设置

### <a name="create-a-translator-resource"></a>创建“翻译”资源

Azure 认知服务由你订阅的 Azure 资源表示。 在本地计算机上使用 [Azure 门户](/cognitive-services/cognitive-services-apis-create-account)或 [Azure CLI](/cognitive-services/cognitive-services-apis-create-account-cli) 创建用于“翻译”的资源。 你还可以：

* 查看 [Azure 门户](https://portal.azure.cn/)中的现有资源。

从试用订阅或资源获取密钥后，创建两个[环境变量](/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication)：

* `TRANSLATOR_TEXT_SUBSCRIPTION_KEY` - “翻译”资源的订阅密钥。
* `TRANSLATOR_TEXT_ENDPOINT` - 翻译器的终结点。 改用 `https://api.translator.azure.cn/`

