---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/22/2020
title: 库 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用和管理库。
ms.openlocfilehash: 0ca4d2df86ff6ef497b44f8d76bca32c31899d67
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937828"
---
# <a name="libraries"></a>库

若要使第三方或自定义代码可供在你的群集上运行的笔记本和作业使用，你可以安装库。 可以用 Python、Java、Scala 和 R 编写库。你可以上传 Java、Scala 和 Python 库，将其指向 PyPI、Maven 和 CRAN 存储库中的外部包。

本文重点介绍了如何在工作区 UI 中执行库任务。 你还可以使用[库 CLI](../dev-tools/cli/libraries-cli.md) 或[库 API](../dev-tools/api/latest/libraries.md) 来管理库。

> [!TIP]
>
> 默认情况下，Databricks 会安装许多常用库。 若要查看默认情况下安装哪些库，请查看适用于 Databricks Runtime 版本的 [Databricks Runtime 发行说明](../release-notes/runtime/releases.md)中的“系统环境”小节。

可以采用以下三种模式之一来安装库：工作区、群集安装，以及作用域为笔记本。

* [工作区库](workspace-libraries.md)充当本地存储库，你可以从中创建群集安装库。 工作区库可能是你的组织创建的自定义代码，也可能是你的组织已经标准化的开源库的特定版本。
* [群集库](cluster-libraries.md)可供群集上运行的所有笔记本使用。 可以直接从公共存储库（例如 PyPI 或 Maven）安装群集库，也可以从以前安装的工作区库中创建一个。
* [作用域为笔记本的 Python 库](notebooks-python-libraries.md)允许你安装 Python 库并创建作用域为笔记本会话的环境。 作用域为笔记本的库不会影响在同一群集上运行的其他笔记本。 这些库不会保留，必须为每个会话重新安装这些库。

  当每个特定笔记本需要一个自定义 Python 环境时，请使用作用域为笔记本的库。 使用作用域为笔记本的库，还可以保存、重用和共享 Python 环境。

  * 在 Databricks Runtime ML 6.4 及更高版本中，可通过 `%pip` 和 `%conda` magic 命令使用作用域为笔记本的库，在Databricks Runtime 7.1 及更高版本中，可通过 `%pip` magic命令使用这些库。 请参阅[作用域为笔记本的 Python 库](notebooks-python-libraries.md)。
  * 在所有 Databricks Runtime 版本中，都可以通过库实用工具使用作用域为笔记本的库。 请参阅[库实用工具](../dev-tools/databricks-utils.md#dbutils-library)。

本部分的内容：

* [工作区库](workspace-libraries.md)
* [群集库](cluster-libraries.md)
* [笔记本范围内的 Python 库](notebooks-python-libraries.md)