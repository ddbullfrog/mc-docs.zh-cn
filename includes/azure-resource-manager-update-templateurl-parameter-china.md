---
title: include 文件
description: include 文件
services: azure-resource-manager
author: rockboyfor
ms.service: azure-resource-manager
ms.topic: include
origin.date: 07/07/2020
ms.date: 09/25/2020
ms.testscope: no
ms.testdate: 09/15/2020
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: c89dcc56fc2cc1a64d837eca4301ecdfebc4e97a
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246724"
---
> [!NOTE]
> 当我们使用以 `https://raw.githubusercontent.com/` 开头的指定模板文件 URI 部署资源时，控制台有时将返回错误，如 `Unable to download deployment content`。
>
> 可以执行以下操作来解决相应问题。
> 1. 下载指定 URI 的模板文件内容并以同一名称另存在本地电脑上。
> 2. 将 `TemplateUri` 的参数替换为 `TemplateFile`，然后用下载的实际文件名更新指定的 URI，并再次运行。
> 
>    | 类别   | 参考链接                   | 操作   |
>    |---         | --------                         |----------|
>    | PowerShell | [New-AzResourceGroupDeployment](https://docs.microsoft.com/powershell/module/Az.Resources/New-AzResourceGroupDeployment) | 将 `-TemplateUri` 替换为 '-TemplateFile` |
>    | Azure CLI  | [az 部署组创建](https://docs.microsoft.com/cli/azure/deployment/group?view=azure-cli-latest#az-deployment-group-create) | 将 `--template-uri` 替换为 '--template-file`|
>