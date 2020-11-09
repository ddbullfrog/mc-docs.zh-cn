---
title: 使用 Azure Stack Hub 门户创建 Windows VM
description: 了解如何使用 Azure Stack Hub 门户创建 Windows Server 2016 虚拟机 (VM)。
author: WenJason
ms.topic: quickstart
origin.date: 10/06/2020
ms.date: 11/09/2020
ms.author: v-jay
ms.reviewer: kivenkat
ms.lastreviewed: 10/06/2020
ms.openlocfilehash: d133e7cebf9af09ad21b326d311b56982b4ce4c9
ms.sourcegitcommit: f187b1a355e2efafea30bca70afce49a2460d0c7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/04/2020
ms.locfileid: "93330501"
---
# <a name="quickstart-create-a-windows-server-vm-with-the-azure-stack-hub-portal"></a>快速入门：使用 Azure Stack Hub 门户创建 Windows 服务器 VM

了解如何使用 Azure Stack Hub 门户创建 Windows Server 2016 虚拟机 (VM)。

> [!NOTE]  
> 本文中的屏幕截图已更新，以匹配 Azure Stack Hub 版本 1808 中引入的用户界面。 除了非托管磁盘外，1808 版还添加了对使用 *托管磁盘* 的支持。 如果使用早期版本，则某些图像（如磁盘选择）将与本文中显示的不同。  


## <a name="sign-in-to-the-azure-stack-hub-portal"></a>登录到 Azure Stack Hub 门户

登录到 Azure Stack Hub 门户。 Azure Stack Hub 门户的地址取决于要连接到的 Azure Stack Hub 产品：

* 对于 Azure Stack 开发工具包 (ASDK)，请转到 `https://portal.local.azurestack.external` 。
* 对于 Azure Stack Hub 集成系统，请转到 Azure Stack Hub 操作员提供的 URL。

## <a name="create-a-vm"></a>创建 VM

1. 选择“创建资源”   > “计算”  。 搜索 ` Windows Server 2016 Datacenter`。

    ![Windows Server 2016 Datacenter - 即用即付](./media/azure-stack-quick-windows-portal/image1a.png)

1. 选择“创建”  。

    ![创建资源](./media/azure-stack-quick-windows-portal/image2a.png)

1. 在“基本信息”下，输入“名称”、“磁盘类型”、“用户名”和“密码”。      选择“订阅”  。 创建一个 **资源组** 或选择现有资源组，选择一个 **位置** ，然后选择“确定”。 

    ![创建 VM - 基本信息](./media/azure-stack-quick-windows-portal/image3a.png)

1. 在“大小”下，选择“D1_v2”，然后选择“选择”。   

    ![创建 VM - 大小](./media/azure-stack-quick-windows-portal/image4a.png)

1. 在“设置”  页上，对默认设置进行任何所需的更改。 你必须从相关下拉框中配置所需的公共入站端口。 完成后，选择“确定”  。

    ![创建 VM - 设置](./media/azure-stack-quick-windows-portal/image5a.png)

1. 在“摘要”下，选择“确定”以创建 VM。  

    ![创建 VM - 摘要](./media/azure-stack-quick-windows-portal/image6a.png)

1. 选择“虚拟机”来查看你的新 VM。  搜索 VM 名称，然后在搜索结果中选择该 VM。

![创建 VM - 搜索 VM](./media/azure-stack-quick-windows-portal/image7a.png)

## <a name="clean-up-resources"></a>清理资源

使用完 VM 后，请删除 VM 及其资源。 为此，请选择 VM 页上的资源组，然后选择“删除”。 

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已部署了一台基本的 Windows Server VM。 若要了解 Azure Stack VM Hub 的详细信息，请继续阅读 [Azure Stack Hub 中 VM 的注意事项](azure-stack-vm-considerations.md)。
