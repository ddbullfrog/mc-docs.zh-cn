---
title: docker run 命令的运行容器示例
titleSuffix: Azure Cognitive Services
description: 适用于情绪分析容器的 docker run 命令
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 10/26/2020
ms.author: v-johya
ms.openlocfilehash: 5385b47403483953954b5c08d70e092282d3948e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105640"
---
若要运行情绪分析 v3 容器，请执行以下 `docker run` 命令。 将下面的占位符替换为你自己的值：

| 占位符 | Value | 格式或示例 |
|-------------|-------|---|
| **{API_KEY}** | 文本分析资源的密钥。 可以在 Azure 门户中资源的“密钥和终结点”页上找到此项。 |`xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`|
| **{ENDPOINT_URI}** | 用于访问文本分析 API 的终结点。 可以在 Azure 门户中资源的“密钥和终结点”页上找到此项。 | `https://<your-custom-subdomain>.cognitiveservices.azure.cn` |

```bash
docker run --rm -it -p 5000:5000 --memory 8g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

此命令：

* 从容器映像运行情绪分析容器
* 分配一个 CPU 核心和 8 GB 内存
* 公开 TCP 端口 5000，并为容器分配伪 TTY
* 退出后自动删除容器。 容器映像在主计算机上仍然可用。

