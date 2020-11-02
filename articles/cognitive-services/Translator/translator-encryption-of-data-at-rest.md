---
title: 翻译器的静态数据加密
titleSuffix: Azure Cognitive Services
description: Microsoft 允许你使用自己的密钥（称为客户管理的密钥 (CMK)）管理你的认知服务订阅。 本文介绍翻译器的静态数据加密，以及如何启用和管理 CMK。
author: Johnnytechn
manager: venkyv
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 10/22/2020
ms.author: v-johya
ms.openlocfilehash: 51908a4ca66754441f390befff2826281961acbe
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473126"
---
# <a name="translator-encryption-of-data-at-rest"></a>翻译器的静态数据加密

将上传的数据持久保存到云中时，翻译器会自动对其加密，以构建自定义的翻译模型，这有助于实现组织的安全性和合规性目标。

## <a name="about-cognitive-services-encryption"></a>关于认知服务加密

数据将使用符合 FIPS 140-2 的 256 位 AES 加密法进行加密和解密。 加密和解密都是透明的，这意味着将替你管理加密和访问。 你的数据默认情况下就是安全的，你无需修改代码或应用程序，即可利用加密。

## <a name="about-encryption-key-management"></a>关于加密密钥管理

默认情况下，订阅使用 Microsoft 托管的加密密钥。
