---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: 从旧区域 URL 迁移到每工作区 URL - Azure Databricks
description: 了解 Azure Databricks 工作区 URL，学习如何开始使用唯一的每工作区 URL。
ms.openlocfilehash: 4216abd36720a56dfcaf6102fae9ce963a7a3270
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937734"
---
# <a name="migrate-from-legacy-regional-to-per-workspace-urls"></a>从旧的区域 URL 迁移到每工作区 URL

2020 年 4 月，Azure Databricks 为每个工作区添加了新的唯一每工作区 URL。 此每工作区 URL 采用以下格式

`adb-<workspace-id>.<random-number>.databricks.azure.cn`

此 URL 是对目前为止用于访问工作区的旧区域 URL (`<region>.databricks.azure.cn`) 的补充。 这两个 URL 都继续受到支持。 但是，随着 Azure Databricks 将更多基础结构添加到现有区域中，新工作区的旧区域 URL 可能与现有工作区的区域 URL 不同。 因此，强烈建议在要用于多个工作区的脚本或其他自动化中使用新的每工作区 URL。

> [!IMPORTANT]
>
> 新工作区不支持旧区域 URL。

## <a name="how-do-i-launch-my-workspace-using-the-per-workspace-url"></a>如何使用每工作区 URL 启动我的工作区？

在 Azure 门户中，转到工作区的 Azure Databricks 服务资源页面，单击“启动工作区”，或者复制此资源页上显示的每工作区 URL 并将其粘贴到浏览器地址栏。

> [!div class="mx-imgBorder"]
> ![资源页](../_static/images/workspace/resource-per-workspace-url.png)

## <a name="migrate-scripts-and-other-automation"></a>迁移脚本和其他自动化

Azure Databricks 用户通常采用下面两种方式之一编写脚本或其他自动化来引用工作区：

* 可在同一区域中创建所有工作区，并在脚本中对旧区域 URL 进行硬编码。

  由于每个工作区都需要一个 API 令牌，因此你还会有一个令牌列表，它存储在脚本本身或其他数据库中。 如果是这种情况，建议存储 `<per-workspace-url, api-token>` 对的列表，并删除所有硬编码的区域 URL。

* 可在一个或多个区域中创建工作区，并将 `<regional-url, api-token>` 对的列表存储在脚本本身或某数据库中。 如果是这种情况，建议将每工作区 URL 而非区域 URL 存储在列表中。

> [!NOTE]
>
> 由于区域 URL 和每工作区 URL 均受支持，因此任何现有自动化（它们使用区域 URL 引用在引入每工作区 URL 之前创建的工作区）都将继续运作。 尽管我们建议你更新所有自动化来使用每工作区 URL，但此情况下无需这样做。

## <a name="find-the-legacy-regional-url-for-a-workspace"></a>查找工作区的旧区域 URL

如果需要查找工作区的旧区域 URL，请在每工作区 URL 上运行 `nslookup`。

```bash
$ nslookup adb-<workspace-id>.<random-number>.databricks.azure.cn
Server:   192.168.50.1
Address:  192.168.50.1#53

Non-authoritative answer:
adb-<workspace-id>.<random-number>.databricks.azure.cn canonical name = chinaeast2-c3.databricks.azure.cn.
Name: chinaeast2-c3.databricks.azure.cn
Address: 20.42.4.211
```