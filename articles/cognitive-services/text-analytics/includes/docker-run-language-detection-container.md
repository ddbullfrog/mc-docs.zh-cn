---
title: docker run 命令的运行容器示例
titleSuffix: Azure Cognitive Services
description: 适用于语言检测容器的 docker run 命令
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 08/03/2020
ms.author: v-johya
ms.openlocfilehash: aa6aec2f94382b4e4a12dff57a70b41dfdc917fe
ms.sourcegitcommit: caa18677adb51b5321ad32ae62afcf92ac00b40b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/08/2020
ms.locfileid: "92211340"
---
若要运行语言检测容器，请执行以下 `docker run` 命令。 将下面的占位符替换为你自己的值：

| 占位符 | Value | 格式或示例 |
|-------------|-------|---|
| **{API_KEY}** | 文本分析资源的密钥。 可以在 Azure 门户中资源的“密钥和终结点”页上找到此项。 |`xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`|
| **{ENDPOINT_URI}** | 用于访问文本分析 API 的终结点。 可以在 Azure 门户中资源的“密钥和终结点”页上找到此项。 | `https://<your-custom-subdomain>.cognitiveservices.azure.com` |


```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/language \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

此命令：

* 从容器映像运行语言检测容器
* 分配一个 CPU 核心和 4 GB 内存
* 公开 TCP 端口 5000，并为容器分配伪 TTY
* 退出后自动删除容器。 容器映像在主计算机上仍然可用。

