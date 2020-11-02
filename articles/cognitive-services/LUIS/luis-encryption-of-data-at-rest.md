---
title: 语言理解服务静态数据加密
titleSuffix: Azure Cognitive Services
description: Microsoft 提供了 Microsoft 托管的加密密钥，还可让你使用自己的密钥（称为客户管理的密钥 (CMK)）管理你的认知服务订阅。 本文介绍语言理解 (LUIS) 的静态数据加密，以及如何启用和管理 CMK。
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 10/19/2020
ms.author: v-johya
ms.openlocfilehash: a1dd4395156a3f4fd12061c8e3e339d62a83d651
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472964"
---
# <a name="language-understanding-service-encryption-of-data-at-rest"></a>语言理解服务静态数据加密

语言理解服务在将数据保存到云时会自动加密数据。 语言理解服务加密可保护数据，并帮助组织履行在安全性与合规性方面做出的承诺。

## <a name="about-cognitive-services-encryption"></a>关于认知服务加密

数据将使用符合 FIPS 140-2 的 256 位 AES 加密法进行加密和解密。 加密和解密都是透明的，这意味着将替你管理加密和访问。 你的数据默认情况下就是安全的，你无需修改代码或应用程序，即可利用加密。

## <a name="about-encryption-key-management"></a>关于加密密钥管理

默认情况下，订阅使用 Microsoft 托管的加密密钥。 

