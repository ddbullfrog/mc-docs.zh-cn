---
title: include 文件
description: include 文件
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
origin.date: 03/29/2019
ms.date: 08/13/2020
ms.author: v-tawe
ms.custom: include file
ms.openlocfilehash: 234a8ab8545f2843f1732a0225ffc7d9e7a84f54
ms.sourcegitcommit: 9d9795f8a5b50cd5ccc19d3a2773817836446912
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/14/2020
ms.locfileid: "92170790"
---
若要通过容器打开直接的 SSH 会话，应用应该处于正在运行状态。

将以下 URL 粘贴到浏览器中，将 \<app-name> 替换为应用名称：

```
https://<app-name>.scm.chinacloudsites.cn/webssh/host
```

如果尚未进行身份验证，则需通过要连接的 Azure 订阅进行身份验证。 完成身份验证以后，可以看到一个浏览器内 shell，可以在其中的容器中运行命令。

![SSH 连接](./media/app-service-web-ssh-connect-no-h/app-service-linux-ssh-connection.png)
