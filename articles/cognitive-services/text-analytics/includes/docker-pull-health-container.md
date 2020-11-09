---
title: 适用于运行状况容器的 docker pull
titleSuffix: Azure Cognitive Services
description: 文本分析的用于运行状况容器的 docker pull 命令
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 10/26/2020
ms.author: v-johya
ms.openlocfilehash: ec8f62a3f879cde71c84c0ab1e09f1583ea2ad46
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105641"
---
填写并提交[认知服务容器请求表单](https://aka.ms/csgate)以请求访问容器。
通过该表单请求有关你、你的公司以及要使用该容器的用户方案的信息。 提交表单后，Azure 认知服务团队可以检查它，确保你满足访问专用容器注册表的条件。

> [!IMPORTANT]
> * 在此表单上，必须使用与 Azure 订阅 ID 关联的电子邮件地址。
> * 用于运行容器的 Azure 资源必须已使用批准的 Azure 订阅 ID 创建。 
> * 请检查你的电子邮件（“收件箱”和“垃圾邮件”文件夹）以获取来自 Microsoft 的应用程序状态更新。

将 docker login 命令与加入电子邮件中提供的凭据结合使用，以连接到认知服务容器的专用容器注册表。


```Docker
docker login containerpreview.azurecr.io -u <username> -p <password>
```

使用 [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) 命令从专用容器注册表下载此容器映像。

```
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-healthcare:latest
```

