---
title: include 文件
description: include 文件
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
origin.date: 03/02/2020
ms.date: 03/30/2020
ms.author: v-tawe
ms.custom: include file
ms.openlocfilehash: 2eefa2a8ea769c59a4a3c5ffee8325c64cc7178d
ms.sourcegitcommit: 44d3fe59952847e5394bbe6c05bd6f333bb56345
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/01/2020
ms.locfileid: "92170789"
---
## <a name="robots933456-in-logs"></a>日志中的 robots933456

你可能会在容器日志中看到以下消息：

```
2019-04-08T14:07:56.641002476Z "-" - - [08/Apr/2019:14:07:56 +0000] "GET /robots933456.txt HTTP/1.1" 404 415 "-" "-"
```

可以放心忽略此消息。 `/robots933456.txt` 是一个虚拟 URL 路径，应用服务使用它来检查容器能否为请求提供服务。 404 响应只是指示该路径不存在，但它让应用服务知道容器处于正常状态并已准备就绪，可以响应请求。

