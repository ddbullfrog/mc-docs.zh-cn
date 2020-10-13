---
title: 将 Azure 门户与 Azure Stack HCI 配合使用
description: 如何使用 Azure 门户查看和管理 Azure Stack HCI 群集。
author: WenJason
ms.author: v-jay
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
origin.date: 09/15/2020
ms.date: 10/12/2020
ms.openlocfilehash: 40d50dc66b09e4d7cee45be1766d62365b4d3951
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91451255"
---
# <a name="use-the-azure-portal-with-azure-stack-hci"></a>将 Azure 门户与 Azure Stack HCI 配合使用

> 适用于：Azure Stack HCI 版本 20H2；Windows Server 2019

本主题说明如何连接到 Azure 门户的 Azure Stack HCI 部分，以获得 Azure Stack HCI 群集的全局视图。 可以使用 Azure 门户管理和监视群集，即使物理基础结构在本地托管也是如此。 基于云的监视无需维护本地监视系统和数据库，从而降低了基础结构复杂性。 它还通过将警报和其他信息直接上传到 Azure（已每天管理数百万个对象）来提高可伸缩性。

## <a name="view-your-clusters-in-the-azure-portal"></a>在 Azure 门户中查看群集

注册 Azure Stack HCI 群集后，其 Azure 资源会显示在 Azure 门户中。 若要查看它，请先登录 [Azure 门户](https://portal.azure.cn)。 如果已[向 Azure 注册了群集](../deploy/register-with-azure.md)，则应该会看到一个新资源组，其中你的群集名称追加了“-rg”。 如果未显示你的 Azure Stack HCI 资源组，请搜索“hci”并从下拉菜单中群集：

:::image type="content" source="media/azure-portal/azure-portal-search.png" alt-text="在 Azure 门户中搜索 hci，以查找 Azure Stack HCI 资源":::

Azure Stack HCI 服务的主页会列出所有群集及其资源组、位置和关联订阅。

:::image type="content" source="media/azure-portal/azure-portal-home.png" alt-text="在 Azure 门户中搜索 hci，以查找 Azure Stack HCI 资源":::

单击 Azure Stack HCI 资源可查看该资源的概述页，其中显示群集和服务器节点的高级摘要。

:::image type="content" source="media/azure-portal/azure-portal-overview.png" alt-text="在 Azure 门户中搜索 hci，以查找 Azure Stack HCI 资源":::

## <a name="view-the-activity-log"></a>查看活动日志

活动日志提供群集上最近操作和事件的列表及其状态、时间、关联订阅和发起用户。 可以按订阅、严重性、时间跨度、资源组和资源筛选事件。

:::image type="content" source="media/azure-portal/azure-portal-activity-log.png" alt-text="在 Azure 门户中搜索 hci，以查找 Azure Stack HCI 资源":::

## <a name="configure-access-control"></a>配置访问控制

使用访问控制可检查用户访问权限、管理角色以及添加和查看角色分配和拒绝分配。

:::image type="content" source="media/azure-portal/azure-portal-iam.png" alt-text="在 Azure 门户中搜索 hci，以查找 Azure Stack HCI 资源":::

## <a name="add-and-edit-tags"></a>添加和编辑标记

标记是名称/值对，可让你通过将相同的标记应用到多个资源和资源组，对资源进行分类并查看合并的账单。 标记名称不区分大小写，但标记值区分大小写。 [了解有关标记的详细信息](/azure-resource-manager/management/tag-resources)。

:::image type="content" source="media/azure-portal/azure-portal-tags.png" alt-text="在 Azure 门户中搜索 hci，以查找 Azure Stack HCI 资源":::

## <a name="compare-azure-portal-and-windows-admin-center"></a>比较 Azure 门户和 Windows Admin Center

与 Windows Admin Center 不同，Azure Stack HCI 的 Azure 门户体验旨在实现全局规模多群集监视。 使用下表可帮助确定适合你需求的管理工具。 它们共同提供了一致的设计，在补充方案中非常有用。

| Windows Admin Center | Azure 门户 |
| --------------- | --------------- |
| 边缘-本地硬件和虚拟机 (VM) 管理，始终可用 | 大规模管理，附加功能 |
| 管理 Azure Stack HCI 基础结构 | 管理其他 Azure 服务 |
| 监视和更新各个群集 | 大规模监视和更新 |
| 手动 VM 预配和管理 | 来自 Azure Arc 的自助服务 VM |

## <a name="next-steps"></a>后续步骤

如需相关信息，另请参阅：

- [将 Azure Stack HCI 连接到 Azure](../deploy/register-with-azure.md)
- [使用 Azure Monitor 监视 Azure Stack HCI](azure-monitor.md)
