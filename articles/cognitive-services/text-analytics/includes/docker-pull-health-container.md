---
title: 适用于运行状况容器的 docker pull
titleSuffix: Azure Cognitive Services
description: 文本分析的用于运行状况容器的 docker pull 命令
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 08/03/2020
ms.author: v-johya
ms.openlocfilehash: eed18510fcd69dffb8f1c2c799354c6f10ee7141
ms.sourcegitcommit: caa18677adb51b5321ad32ae62afcf92ac00b40b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/08/2020
ms.locfileid: "92211346"
---
填写并提交[认知服务容器请求表单](https://aka.ms/cognitivegate)以请求访问容器。

[!INCLUDE [Request access to the container registry](../../../../includes/cognitive-services-containers-request-access-only.md)]

将 docker login 命令与加入电子邮件中提供的凭据结合使用，以连接到认知服务容器的专用容器注册表。

```bash
docker login containerpreview.azurecr.io -u <username> -p <password>
```

使用 [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) 命令从专用容器注册表下载此容器映像。

```
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-healthcare:latest
```

