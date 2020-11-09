---
title: 使用 Azure 门户部署 Azure 专用主机
description: 使用 Azure 门户将 VM 和规模集部署到专用主机。
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure
origin.date: 09/04/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: 5d51a18a0433856e774f071f5ffa4b25971e8b8e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106137"
---
---
<!--Verified successfully-->
<!--Verified the Portal UI and confirmed with Peter Gu-->
# <a name="deploy-vms-to-dedicated-hosts-using-the-portal"></a>使用门户将 VM 部署到专用主机

本文介绍了如何创建 Azure [专用主机](dedicated-hosts.md)来托管虚拟机 (VM)。 

<!--Not Available on Automatic placement of VMs and scale set instances till on 10/27/2020-->
<!--Verified on Portal-->

## <a name="limitations"></a>限制

- 专用主机可用的大小和硬件类型因区域而异。 请参阅主机[定价页](https://www.azure.cn/pricing/details/virtual-machines/)来了解详细信息。

<!--MOONCAKE CUSTOMIZATION ON 05/22/2020-->

## <a name="create-a-host-group"></a>创建主机组

主机组是表示专用主机集合的资源。 你在某个区域中创建主机组，并向其中添加主机。 规划高可用性时，可以为专用主机进行以下选择： 

<!--Not Available on there are additional options.-->
<!--Not Available on one or both of-->
<!--Not Available on and an availability zone-->

<!--Not Available on - Span across multiple availability zones.-->

- 跨映射到物理机架的多个容错域。 
 
在这种情况下，你需要为主机组提供容错域计数。

<!--Not Available on  If you do not want to span fault domains in your group, use a fault domain count of 1.-->
<!--Not Available on You can also decide to use both availability zones and fault domains. -->

在此示例中，我们将创建使用 2 个容错域的主机组。 

<!--CORRECT TO REMOVE 1 availability zone and-->
<!--Not Available on If you would like to try the preview for **Automatic placement**, use this URL: [https://aka.ms/vmssadh](https://aka.ms/vmssadh)-->

1. 打开 [Azure 门户](https://portal.azure.cn)。
1. 选择左上角的“创建资源”。
1. 搜索“主机组”，然后从结果中选择“主机组”。
1. 在“主机组”页中，选择“添加”。
1. 选择要使用的订阅，然后选择“新建”以创建新的资源组。
1. 键入“myDedicatedHostsRG”作为“名称”，然后选择“确定”。
1. 对于“主机组名称”，请键入“myHostGroup”。
1. 对于“位置”，请选择“中国东部”。 
    
    <!--Not Available on 1. For **Availability Zone**, select **1**.-->
    
1. 选择 **2** 作为“容错域计数”。

    <!--Not Available on  **Automatic placement** URL till on 10/27/2020-->
    
1. 选择“查看 + 创建”，然后等待验证。
1. 看到“验证通过”消息后，选择“创建”以创建主机组。 

应当只需很短时间便可创建主机组。

<!--MOONCAKE CUSTOMIZATION ON 05/22/2020-->

## <a name="create-a-dedicated-host"></a>创建专用主机

现在，在主机组中创建一个专用主机。 除了主机名称外，还需要提供主机的 SKU。 主机 SKU 捕获受支持的 VM 系列以及专用主机的硬件代系。

有关主机 SKU 和定价的详细信息，请参阅 [Azure 专用主机定价](https://www.azure.cn/pricing/details/virtual-machines/)。

如果为主机组设置了容错域计数，则系统会要求你为主机指定容错域。  

1. 选择左上角的“创建资源”。
    
    <!--MOONCAKE CUSTOMIZED ON 10/27/2020-->
    
1. 搜索“主机”，然后从结果中选择“主机”。
1. 在“主机”页中，选择“添加”。 
    
    <!--MOONCAKE CUSTOMIZED ON 10/27/2020-->
    
1. 选择要使用的订阅。
1. 选择“myDedicatedHostsRG”作为“资源组”。
1. 在“实例详细信息”中，键入“myHost”作为“名称”，并选择“中国东部”作为位置。
1. 在“硬件配置文件”中，对于“大小系列”，请选择“标准 Es3 系列 - 类型 1”；对于“主机组”，请选择“myHostGroup”；对于“容错域”，请选择 1。 至于其余字段，请保留默认值。
1. 完成后，选择“查看 + 创建”，然后等待验证。
1. 看到“验证通过”消息后，选择“创建”以创建主机。 

## <a name="create-a-vm"></a>创建 VM

1. 在 Azure 门户的左上角，选择“创建资源”。
1. 在 Azure 市场资源列表上方的搜索框中，搜索并选择要使用的映像，然后选择“创建”。
1. 在“基本信息”选项卡中的“项目详细信息”下，确保选择了正确的订阅，然后选择 myDedicatedHostsRG 作为“资源组” 。 
1. 在“实例详细信息”下，对于“虚拟机名称”，键入“myVM”，对于“位置”，选择“中国东部”。
    
    <!--Not Available on **Availability options** select **Availability zone**-->
    
1. 对于大小，选择“更改大小”。 在可用大小列表中，选择 Esv3 系列其中一个，例如“标准 E2s v3”。 可能需要清除筛选器才能查看所有可用大小。
1. 根据需要完成“基本信息”选项卡上的其余字段。
1. 在页面顶部，选择“高级”选项卡，然后在“主机”部分，对于“主机组”，选择 myHostGroup，对于“主机”，选择 myHost 。 
    :::image type="content" source="./media/dedicated-hosts-portal/advanced.png" alt-text="选择主机组和主机":::
1. 保留剩余的默认值，然后选择页面底部的“查看 + 创建”按钮。
1. 显示验证通过的消息时，选择“创建”。

部署 VM 需要数分钟。

<!--Not Available on ## Create a scale set (preview)-->
<!--Not Available on 1. On the **Advanced** tab, for **Spreading algorithm** select **Max spreading**.-->
<!--Not Availble on Spreading algorithm option till on 10/27/2020-->

## <a name="add-an-existing-vm"></a>添加现有 VM 

可将现有 VM 添加到专用主机，但必须先停止/解除分配该 VM。 在将 VM 移动到专用主机之前，请确保 VM 配置受支持：

- VM 大小必须属于专用主机所用的同一大小系列。 例如，如果专用主机是 DSv3，则 VM 大小可以是 Standard_D4s_v3，但不能是 Standard_A4_v2。 
- VM 需要位于专用主机所在的同一区域。
- VM 不能是邻近放置组的一部分。 在将 VM 移动到专用主机之前，请先从邻近放置组中删除该 VM。 有关详细信息，请参阅[将 VM 移出邻近放置组](./windows/proximity-placement-groups.md#move-an-existing-vm-out-of-a-proximity-placement-group)
- VM 不能位于可用性集中。

    <!--Not Available on  availability zone-->

使用[门户](https://portal.azure.cn)将 VM 迁移到专用主机。

1. 打开 VM 所对应的页。
1. 选择“停止”以停止/解除分配 VM。
1. 在左侧菜单中选择“配置”。
1. 从下拉菜单中选择主机组和主机。
1. 完成操作后，在页面顶部选择“保存”。
1. 将 VM 添加到主机之后，从左侧菜单中选择“概述”。
1. 在页面顶部，选择“启动”以重启 VM。

## <a name="next-steps"></a>后续步骤

- 有关详细信息，请参阅[专用主机](dedicated-hosts.md)概述。

- [此处](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md)有一个示例模板，该模板使用区域和容错域来最大限度地提高在某个地区的复原能力。

- 还可以使用 [Azure CLI](./linux/dedicated-hosts-cli.md) 或 [PowerShell](./windows/dedicated-hosts-powershell.md) 部署专用主机。

<!-- Update_Description: update meta properties, wording update, update link -->
