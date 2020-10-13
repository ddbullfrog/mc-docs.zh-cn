---
title: 使用 Windows Admin Center 创建 Kubernetes 群集的快速入门
description: 了解如何使用 Windows Admin Center 创建 Kubernetes 群集
author: WenJason
ms.service: azure-stack
ms.topic: quickstart
origin.date: 09/22/2020
ms.date: 10/12/2020
ms.author: v-jay
ms.openlocfilehash: 2027413ddc626055a062376b4e5fcd7785160d91
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91451201"
---
# <a name="quickstart-create-a-kubernetes-cluster-on-azure-stack-hci-using-windows-admin-center"></a>快速入门：使用 Windows Admin Center 在 Azure Stack HCI 上创建 Kubernetes 群集

> 适用于：Azure Stack HCI

设置 Azure Kubernetes 服务主机之后，可以使用 Windows Admin Center 创建 Kubernetes 群集。 若要改为使用 PowerShell，请参阅[使用 PowerShell 创建 Kubernetes 群集](create-kubernetes-cluster-powershell.md)。

让我们开始吧：

1. [设置 Azure Kubernetes 服务](setup.md)并检查[系统要求](system-requirements.md)（如果尚未这样做）。
1. 若要开始在 Windows Admin Center 中创建 Kubernetes 群集，请按网关屏幕上的“添加”按钮。
2. 在“添加或创建资源”面板中的“Kubernetes 群集(预览)”下，选择“新建”以启动 Kubernetes 群集向导  。 尽管 Kubernetes 群集下的“添加”按钮在公共预览版中提供，但它不起作用。 在 Kubernetes 群集创建过程中的任何时候，都可以退出向导，但请注意，不会保存进度。 

    ![说明 Windows Admin Center 中的“添加或创建资源”边栏选项卡（现在包含用于 Kubernetes 群集的新磁贴）。](.\media\create-kubernetes-cluster\add-connection.png)

3. 查看将托管 Kubernetes 群集的系统的先决条件，以及 Windows Admin Center 的先决条件。 完成后，选择“下一步”。 
4. 在“基本信息”页上，配置有关群集的信息，例如 Azure Kubernetes 服务主机信息和主节点池大小。  Azure Kubernetes 服务主机信息和主节点池大小。 Azure Kubernetes 服务主机字段需要你要将 Kubernetes 群集部署到的 Azure Stack HCI 群集的完全限定的域名。 必须通过 Azure Kubernetes 服务工具为此系统完成主机设置。 在公共预览版中，节点计数不可编辑，会默认为 2，但可以为更大的工作负载配置节点大小。 完成后，选择“下一步”。

    ![说明 Kubernetes 群集向导的“基本信息”页。](.\media\create-kubernetes-cluster\basics.png)

5. 可以在“节点池”页上配置其他节点池来运行工作负载。 在公共预览版期间，可以最多添加一个 Windows 节点池和一个 Linux 节点池（除了系统节点池之外）。 完成后，选择“下一步”****。
6. 在“网络”页上指定网络配置。 如果选择“高级”，则可以自定义在为群集中的节点创建虚拟网络时使用的地址范围。 如果选择“基本”，则会使用默认地址范围创建群集中节点的虚拟网络。 无法在公共预览版中更改网络设置（负载均衡器、网络策略和 HTTP 应用程序路由）。 完成后，选择“下一步”。

    ![说明 Kubernetes 群集向导的“网络”页。](.\media\create-kubernetes-cluster\networking.png)

7. 在“集成”页上，将群集与其他服务（如 Kubernetes 仪表板、持久存储和 Azure Monitor）连接。 持久存储是需要在此页上配置的唯一服务。 在公共预览版中，持久存储位置默认为在主机设置过程中选择的存储位置。 此字段当前不可编辑。 完成后，选择“下一步”****。
8. 在“审阅 + 创建”页上审阅进行的选择。 如果感到满意，请选择“创建”以开始部署。 部署进度会显示在此页的顶部。 
9. 部署完成之后，“后续步骤”页会详细说明如何管理群集。 此页还包含 SSH 密钥和 Kubernetes 仪表板令牌。 如果在上一步中选择禁用 Kubernetes 仪表板或 Azure Arc 集成，则此页上的某些信息和说明可能不可用或不起作用。

> [!IMPORTANT] 
> 建议将 SSH 密钥和 Kubernetes 仪表板令牌保存在安全的可访问位置。

在本快速入门中，你设置了 Azure Kubernetes 服务主机并部署了 Kubernetes 群集。 

若要详细了解 Azure Stack HCI 上的 Azure Kubernetes 服务，并演练如何在 Azure Stack HCI 上的 Azure Kubernetes 服务上部署和管理 Linux 应用程序，请继续学习本教程。
