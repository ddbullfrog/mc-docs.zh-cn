---
title: 管理 Azure 数据资源管理器群集中的语言扩展。
description: 使用语言扩展在 Azure 数据资源管理器 KQL 查询中集成其他语言。
author: orspod
ms.author: v-tawe
ms.reviewer: orhasban
ms.service: data-explorer
ms.topic: how-to
origin.date: 04/01/2020
ms.date: 09/24/2020
ms.openlocfilehash: e2ff43009e9e3c540d8eee4fc507d281534f58fd
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146355"
---
# <a name="manage-language-extensions-in-your-azure-data-explorer-cluster-preview"></a>管理 Azure 数据资源管理器群集（预览版）中的语言扩展

语言扩展功能允许你使用语言扩展插件将其他语言集成到 Azure 数据资源管理器 KQL 查询中。 当你使用相关脚本运行用户定义函数 (UDF) 时，该脚本会获取表格数据作为其输入，并会生成表格输出。 插件运行时托管在[沙盒](kusto/concepts/sandboxes.md)中，这是一个运行在群集节点上的独立且安全的环境。 本文介绍如何在 Azure 门户中管理 Azure 数据资源管理器群集中的语言扩展插件。

> [!NOTE]
> 当前支持的 Azure 数据资源管理器语言扩展为 Python 和 R。

## <a name="prerequisites"></a>先决条件

* 如果没有 Azure 订阅，请在开始前创建一个[试用 Azure 帐户](https://www.azure.cn/pricing/1rmb-trial/)。
* 创建 [Azure 数据资源管理器群集和数据库](create-cluster-database-portal.md)。

## <a name="enable-language-extensions-on-your-cluster"></a>在群集上启用语言扩展

> [!WARNING]
> 启用语言扩展之前，请查看[限制](#limitations)。

执行以下步骤，在群集上启用语言扩展：

1. 在 Azure 门户中，转到 Azure 数据资源管理器群集。 
1. 在“设置”中选择“配置”。  
1. 在“配置”窗格中，选择“打开”以启用某个语言扩展。 
1. 选择“保存” 。
 
    ![启用语言扩展](media/language-extensions/configurations-enable-extension.png)

> [!NOTE]
> 启用语言扩展过程可能需要几分钟时间。 在此期间，群集会暂停。
 
## <a name="run-language-extension-integrated-queries"></a>运行集成了语言扩展的查询

* 了解如何[运行集成了 Python 的 KQL 查询](kusto/query/pythonplugin.md)。
* 了解如何[运行集成了 R 的 KQL 查询](kusto/query/rplugin.md)。 

## <a name="disable-language-extensions-on-your-cluster"></a>在群集上禁用语言扩展

> [!NOTE]
> 禁用语言扩展可能需要几分钟时间。

执行以下步骤，在群集上禁用语言扩展：

1. 在 Azure 门户中，转到 Azure 数据资源管理器群集。 
1. 在“设置”中选择“配置”。  
1. 在“配置”窗格中，选择“关闭”以禁用某个语言扩展。 
1. 选择“保存” 。

    ![禁用语言扩展](media/language-extensions/configurations-disable-extension.png)

## <a name="limitations"></a>限制

* 语言扩展功能不支持[磁盘加密](cluster-disk-encryption.md)。 
* 即使在相关语言的范围内没有查询运行，语言扩展运行时沙盒也会分配磁盘空间。
若要更详细地了解限制，请参阅[沙盒](kusto/concepts/sandboxes.md)。
