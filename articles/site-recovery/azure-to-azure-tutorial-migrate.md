---
title: 使用 Azure Site Recovery 将 Azure VM 移动到不同 Azure 区域
description: 使用 Azure Site Recovery 将 Azure VM 从一个 Azure 区域移动到另一个 Azure 区域。
services: site-recovery
ms.service: site-recovery
ms.topic: tutorial
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 09/07/2020
ms.author: v-yeche
ms.custom: MVC
ms.openlocfilehash: 7199d0b694fcbb9ff93aa64fb54e461f433dccc1
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105076"
---
# <a name="move-vms-to-another-azure-region"></a>将 VM 移动到另一个 Azure 区域

在某些场景中，你希望将现有 Azure IaaS 虚拟机 (VM) 从一个区域移动到另一个区域。

<!--Not Available on  For example, you want to improve reliability and availability of your existing VMs, to improve manageability, or to move for governance reasons. For more information, see the [Azure VM move overview](azure-to-azure-move-overview.md)-->

可以使用 [Azure Site Recovery](site-recovery-overview.md) 服务将 Azure VM 移动到次要区域。

在本教程中，你将了解如何执行以下操作：

> [!div class="checklist"]
> 
> * 验证进行移动的先决条件
> * 准备源 VM 和目标区域
> * 复制数据和启用复制
> * 测试配置和执行移动
> * 删除源区域中的资源

<!--Not Available on [!IMPORTANT]-->
<!--Not Available on [Azure Resource Mover](../resource-mover/tutorial-move-region-virtual-machines.md)-->
> [!NOTE]
> 本教程演示如何将 Azure VM 按原样从一个区域移到另一个区域。

<!--Not Available on [Move Azure VMs into Availability Zones tutorial](move-azure-vms-avset-azone.md)-->

<a name="verify-prerequisites"></a>
## <a name="prerequisites"></a><a name="prerequisites"></a>先决条件

- 请确保 Azure VM 位于要从中移动的 Azure 区域中。
- 验证所选的[源区域 - 目标区域组合是否受支持](./azure-to-azure-support-matrix.md#region-support)，并在目标区域方面做出明智的决策。
- 请确保了解[方案体系结构和组件](azure-to-azure-architecture.md)。
- 查看[支持限制和要求](azure-to-azure-support-matrix.md)。
- 验证帐户权限。 如果你创建了免费 Azure 帐户，那么你就是订阅的管理员。 如果你不是订阅管理员，请要求管理员分配你所需的权限。 若要为 VM 启用复制，并使用 Azure Site Recovery 按原样复制数据，必须：

    - 在 Azure 资源中创建 VM 的权限。 “虚拟机参与者”内置角色具有这些权限，这包括：
    - 在所选资源组中创建 VM 的权限
    - 在所选虚拟网络中创建 VM 的权限
    - 向所选存储帐户进行写入的权限
    - 管理 Azure Site Recovery 操作的权限。 “Site Recovery 参与者”角色拥有管理恢复服务保管库中 Site Recovery 操作所需的全部权限。

- 请确保要移动的 Azure VM 上存在所有最新的根证书。 如果最新的根证书不在 VM 上，则安全性约束将阻止数据复制到目标区域。

- 对于 Windows VM，请在 VM 上安装所有最新的 Windows 更新，使所有受信任的根证书位于该计算机上。 在未联网的环境中，请按照你的组织的标准 Windows 更新和证书更新过程执行操作。

- 对于 Linux VM，请遵循 Linux 分销商提供的指导，在 VM 上获取最新的受信任根证书和证书吊销列表。

- 确保未使用身份验证代理来控制要移动的 VM 的网络连接。

- 如果尝试移动的 VM 无法访问 Internet，或使用防火墙代理来控制出站访问，请[检查要求](azure-to-azure-tutorial-enable-replication.md#set-up-outbound-network-connectivity-for-vms)。

- 确定源网络布局和当前正在使用的所有资源。 这包括但不限于负载均衡器、网络安全组 (NSG) 和公共 IP。

- 验证 Azure 订阅是否允许在用于灾难恢复的目标区域中创建 VM。 请联系支持部门，启用所需配额。

- 请确保订阅中有足够的资源，能够支持大小与源 VM 匹配的 VM。 如果使用 Site Recovery 将数据复制到目标区域，Site Recovery 会为目标 VM 选择相同的大小或尽可能接近的大小。

- 请确保为源网络布局中标识的每个组件创建目标资源。 此步骤对于确保目标区域中具有与源区域中相同的所有功能很重要。

    > [!NOTE]
    > 为源 VM 启用复制时，Azure Site Recovery 会自动发现并创建虚拟网络。 此外可以预先创建网络，并将其分配到用户流中的 VM 以启用复制。 按后文所述，需要在目标区域中手动创建其他资源。

    若要根据源 VM 配置创建最常用的相关网络资源，请参阅以下文档：
    - [网络安全组](../virtual-network/manage-network-security-group.md)
    - [负载均衡器](../load-balancer/index.yml)
    - [公共 IP](../virtual-network/virtual-network-public-ip-address.md)
    - 对于其他任何网络组件，请参阅[网络文档](https://docs.azure.cn/zh-cn/?pivot=products&panel=network)。

    <!--CORRECT ON https://docs.azure.cn/zh-cn/?pivot=products&panel=network-->


## <a name="prepare"></a>准备
以下步骤演示如何使用 Azure Site Recovery 作为解决方案来准备要移动的虚拟机。 

### <a name="create-the-vault-in-any-region-except-the-source-region"></a>在除了源区域之外的任意区域中创建保管库

1. 登录到 [Azure 门户](https://portal.azure.cn)
1. 在搜索中，键入 `Recovery Services` > 单击“恢复服务保管库”
1. 在“恢复服务保管库”菜单中，单击“+添加”。
1. 在“名称”  中，指定友好名称 **ContosoVMVault** 。 如果有多个订阅，请选择合适的一个。
1. 创建资源组 ContosoRG  。
1. 指定 Azure 区域。 若要查看受支持的区域，请参阅 [Azure Site Recovery 定价详细信息](https://www.azure.cn/pricing/details/site-recovery/)中的“地域可用性”。
1. 在“恢复服务保管库”中，选择“ContosoVMVault” > “复制的项” > “+ 复制”   。
1. 在下拉列表中，选择“Azure 虚拟机”。
1. 在“源位置”中，选择当前运行 VM 的 Azure 源区域。
1. 选择“资源管理器”部署模型。 然后选择“源订阅”和“源资源组”。
1. 选择“确定”以保存设置。

### <a name="enable-replication-for-azure-vms-and-start-copying-the-data"></a>为 Azure VM 启用复制并开始复制数据

Site Recovery 会检索与订阅和资源组关联的 VM 列表。

1. 在下一步中，选择要移动的 VM，然后选择“确定”  。
1. 在“设置”中，选择“灾难恢复”   。
1. 在“配置灾难恢复” > “目标区域”中，选择要复制到的目标区域。
1. 对于本教程，接受其他默认设置。
1. 选择“启用复制”  。 此步骤将启动用于为 VM 启用复制的作业。

    :::image type="content" source="media/tutorial-migrate-azure-to-azure/settings.png" alt-text="启用复制":::

## <a name="move"></a>移动

以下步骤演示如何执行移动到目标区域。

1. 转到保管库。 在“受保护的项”   > “复制的项”  中选择 VM，然后选择“故障转移”  。

    <!--MOONCAKE: **Protected Items** to replace **Setting**-->

2. 在“故障转移”  中，选择“最新”  。
3. 选择“在开始故障转移前关闭计算机”  。 Site Recovery 在触发故障转移之前会尝试关闭源 VM。 即使关机失败，故障转移也仍会继续。 可以在“作业”  页上跟踪故障转移进度。
4. 该作业完成后，检查 VM 是否按预期显示在目标 Azure 区域中。

## <a name="discard"></a>弃用 

如果已检查移动的 VM 并且需要更改为故障转移点，或者想要返回到以前的点，请在“复制的项”  中，右键选择“VM”>“更改恢复点”  。 此步骤提供了指定其他恢复点并故障转移到该恢复点的选项。 

## <a name="commit"></a>Commit 

检查了移动的 VM 并准备好提交更改后，请在“复制的项”  中，右键选择“VM”>“提交”  。 此步骤会完成移到目标区域的过程。 请等待提交作业完成。

## <a name="clean-up"></a>清理

以下步骤将指导你完成如何清理源区域以及用于移动的相关资源。

对于用于移动的所有资源：

- 转到 VM。 选择“禁用复制”  。 此步骤会停止复制 VM 数据的过程。

    > [!IMPORTANT]
    > 请务必执行此步骤，避免 Azure Site Recovery 复制产生费用。

如果不打算再次使用任何源资源，请完成以下附加步骤：

1. 删除在[先决条件](#prerequisites)中标识的源区域中的所有相关网络资源。
1. 删除源区域中的相应存储帐户。

## <a name="next-steps"></a>后续步骤

在本教程中，你将 Azure VM 移动到了一个不同的 Azure 区域。 现在，你可以为移动的 VM 配置灾难恢复。

> [!div class="nextstepaction"]
> [在迁移后设置灾难恢复](azure-to-azure-quickstart.md)

<!-- Update_Description: update meta properties, wording update, update link -->