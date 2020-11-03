---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/02/2020
title: 管理虚拟网络 - Azure Databricks
description: 了解如何管理虚拟网络。
ms.openlocfilehash: ae96582933a11e691e41e338a906ac6c06d0326a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106483"
---
# <a name="manage-virtual-networks"></a>管理虚拟网络

默认情况下，每个 Azure Databricks 部署都会在 Azure 订阅中创建一个锁定的[虚拟网络 (VNet)](/virtual-network/virtual-networks-overview)。 所有群集都在该虚拟网络中创建。

> [!div class="mx-imgBorder"]
> ![Azure Databricks 网络体系结构](../../../_static/images/vnet/adb-arch-diagram.png)

可能需要自定义此网络基础结构，包括：

* 将 Azure Databricks 虚拟网络与另一个虚拟网络（不管是远程的还是本地的）对等互连。
* 将 Azure Databricks 的客户托管资源部署在你自己的虚拟网络中（有时称为 VNet 注入）。

以下文章介绍了这些网络自定义的优势以及如何执行它们：

* [将虚拟网络对等互连](vnet-peering.md)
* [在 Azure 虚拟网络中部署 Azure Databricks（VNet 注入）](vnet-inject.md)
* [将“VNet 注入”预览版工作区升级到正式发行版](vnet-inject-upgrade.md)
* [将 Azure Databricks 工作区连接到本地网络](on-prem-network.md)
* [用户定义的 Azure Databricks 路由设置](udr.md)