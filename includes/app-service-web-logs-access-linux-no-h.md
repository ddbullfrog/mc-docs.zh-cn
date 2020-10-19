---
title: include 文件
description: include 文件
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
origin.date: 03/27/2019
ms.date: 08/13/2020
ms.author: v-tawe
ms.custom: include file
ms.openlocfilehash: d4a7a3dd2a4df70bd1fea0d921a5c772d66b8d9e
ms.sourcegitcommit: 9d9795f8a5b50cd5ccc19d3a2773817836446912
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/14/2020
ms.locfileid: "92170827"
---
可以访问在容器中生成的控制台日志。 首先，请在 Cloud Shell 中运行以下命令，以便启用容器日志记录功能：

```azurecli
az webapp log config --name <app-name> --resource-group myResourceGroup --docker-container-logging filesystem
```

启用容器日志记录功能以后，请运行以下命令来查看日志流：

```azurecli
az webapp log tail --name <app-name> --resource-group myResourceGroup
```

如果没有立即看到控制台日志，请在 30 秒后重新查看。

> [!NOTE]
> 也可通过浏览器在 `https://<app-name>.scm.chinacloudsites.cn/api/logs/docker` 中检查日志文件。

若要随时停止日志流式处理，请键入 `Ctrl`+`C`。
