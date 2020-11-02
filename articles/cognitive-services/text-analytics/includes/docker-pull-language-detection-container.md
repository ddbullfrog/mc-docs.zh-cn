---
title: 适用于语言检测容器的 docker pull
titleSuffix: Azure Cognitive Services
description: 适用于语言检测容器的 docker pull 命令
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 10/26/2020
ms.author: v-johya
ms.openlocfilehash: e5a99fb5171a4331cc414eab3e242e11ff4b70b4
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105637"
---
#### <a name="docker-pull-for-the-language-detection-container"></a>适用于语言检测容器的 docker pull

使用 [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) 命令从 Microsoft 容器注册表下载容器映像。

有关文本分析容器的可用标记的完整说明，请参阅 Docker Hub 上的[语言检测](https://go.microsoft.com/fwlink/?linkid=2018759)容器。

```
docker pull mcr.microsoft.com/azure-cognitive-services/textanalytics/language:latest
```

