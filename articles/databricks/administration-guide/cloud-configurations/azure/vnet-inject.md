---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/10/2020
title: 在 Azure 虚拟网络中部署 Azure Databricks（VNet 注入）- Azure Databricks
description: 了解如何在 Azure 虚拟网络中部署 Azure Databricks（VNet 注入）。
ms.openlocfilehash: 7a59d1752f7c82d081885035a7d6fd4ae9b7103e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106519"
---
# <a name="deploy-azure-databricks-in-your-azure-virtual-network-vnet-injection"></a>在 Azure 虚拟网络中部署 Azure Databricks（VNet 注入）

> [!IMPORTANT]
>
> 如果在 VNet 注入预览期间，在自己的 Azure 虚拟网络中部署了 Azure Databricks 工作区，并且尚未升级到正式发行版，在 6 月 1 日将失去对工作区的访问权限。 在 6 月 1 日当天或之后，必须遵照升级步骤操作，然后打开支持工单才能重新访问工作区。 请参阅[将“VNet 注入”预览版工作区升级到正式发行版](vnet-inject-upgrade.md#vnet-inject-upgrade)。

Azure Databricks 的默认部署是 Azure 上的完全托管服务：所有数据平面资源（包括与所有群集关联的[虚拟网络 (VNet)](/virtual-network/virtual-networks-overview)）都部署到锁定的资源组。 但如果需要自定义网络，则可以在自己的虚拟网络（有时称为 VNet 注入）中部署 Azure Databricks 数据平面资源，以便能够：

* 使用[服务终结点](/virtual-network/virtual-network-service-endpoints-overview)以更安全的方式将 Azure Databricks 连接到其他 Azure 服务（如 Azure 存储）。
* 连接到[本地数据源](on-prem-network.md)以与 Azure Databricks 配合使用，从而利于[用户定义的路由](udr.md)。
* 将 Azure Databricks 连接到[网络虚拟设备](on-prem-network.md#route-via-firewall)以检查所有出站流量并根据允许和拒绝规则执行操作。
* 将 Azure Databricks 配置为使用[自定义 DNS](on-prem-network.md#vnet-custom-dns)。
* 配置[网络安全组 (NSG) 规则](/virtual-network/manage-network-security-group)以指定出口流量限制。
* 在现有虚拟网络中部署 Azure Databricks 群集。

通过将 Azure Databricks 数据平面资源部署到自己的虚拟网络中，还可以利用灵活的 CIDR 范围（虚拟网络的 CIDR 范围在 /16-/24 之间，子网最高可达 /26）。

> [!IMPORTANT]
>
> 不能替换现有工作区的虚拟网络。 如果当前工作区无法容纳所需数量的活动群集节点，我们建议在较大的虚拟网络中另外创建一个工作区。 按照这些[详细的迁移步骤](/azure-databricks/howto-regional-disaster-recovery#detailed-migration-steps)将资源（笔记本、群集配置、作业）从旧工作区复制到新工作区。

## <a name="virtual-network-requirements"></a><a id="virtual-network-requirements"> </a><a id="vnet-inject-reqs"> </a> 虚拟网络要求

将 Azure Databricks 工作区部署到其中的虚拟网络必须满足以下要求：

* **位置：** 该虚拟网络必须与 Azure Databricks 工作区位于同一位置。
* **订阅：** 虚拟网络必须与 Azure Databricks 工作区位于同一订阅中。
* **子网：** 虚拟网络必须包含两个专用于 Azure Databricks 的子网：专用子网和公共子网。 公共子网允许与 Azure Databricks 控制平面进行通信。 专用子网只允许群集内部通信。 不要在 Azure Databricks 工作区使用的子网上部署其他 Azure 资源。 与其他资源（如虚拟机）共享子网会阻止对子网的意向策略进行托管更新。

  [在 Azure 门户中创建 Azure Databricks 工作区](#vnet-inject-portal)时，Azure Databricks 会为你创建这些要求，并在工作区部署期间将公共和专用子网委托给 `Microsoft.Databricks/workspaces` 服务，使 Azure Databricks 能够创建[网络安全组规则](#nsg)。 如果需要添加或更新 Azure Databricks 托管的 NSG 规则的范围，Azure Databricks 将始终提前通知。 如果使用[全功能](#arm-all-in-one) Azure 资源管理器模板或[仅限 VNet](#arm-vnet-only)的 Azure 资源管理器模板，则 Azure Databricks 还会为你创建子网。 但如果使用自定义 Azure 资源管理器模板或[工作区](#arm-workspace) Azure 资源管理器模板，则由你来确保子网已附加并正确委托了网络安全组。 有关委托说明，请参阅[将“VNet 注入”预览版工作区升级到正式发行版](vnet-inject-upgrade.md#vnet-inject-upgrade)或[添加或删除子网委托](/virtual-network/manage-subnet-delegation)。

  > [!IMPORTANT]
  >
  > 这些子网和 Azure Databricks 工作区之间存在一对一关系。 不能在单个子网中共享多个工作区。 部署的每个工作区必须拥有一对新的公共和专用子网。

* **地址空间：** 虚拟网络为 /16-/24 之间的 CIDR 块，专用子网和公共子网为最高可达 /26 的 CIDR 块。

可以使用 [Azure 门户中的 Azure Databricks 工作区部署接口](#vnet-inject-portal)来为现有虚拟网络自动配置所需子网，也可以使用 [Azure Databricks 提供的 Azure 资源管理器模板](#vnet-inject-advanced)来配置虚拟网络和部署工作区。

## <a name="create-the-azure-databricks-workspace-in-the-azure-portal"></a><a id="create-the-azure-databricks-workspace-in-the-azure-portal"> </a><a id="vnet-inject-portal"> </a> 在 Azure 门户中创建 Azure Databricks 工作区

本部分介绍如何在 Azure 门户中创建 Azure Databricks 工作区，并将其部署到自己现有的虚拟网络中。 Azure Databricks 使用你提供的 CIDR 范围内的两个新的子网和网络安全组更新虚拟网络，将入站和出站子网流量列入允许列表，并将工作区部署到更新的虚拟网络。

> [!NOTE]
>
> 如果想要更多地控制虚拟网络的配置（例如，你可能想要使用现有子网、使用现有网络安全组，或者创建自己的安全规则），则可以使用 Azure-Databricks 提供的 Azure 资源管理器模板替代门户 UI。 请参阅[配置虚拟网络](#vnet-inject-advanced)。

### <a name="requirements"></a>要求

必须[配置将部署 Azure Databricks 工作区的虚拟网络](/virtual-network/)：

* 可以使用现有的虚拟网络，也可以创建新的虚拟网络，但是虚拟网络必须与你计划创建的 Azure Databricks 工作区位于同一区域和订阅中。
* 虚拟网络需要介于 /16-/24 之间的 CIDR 范围。

  > [!WARNING]
  >
  > 具有较小虚拟网络的工作区比具有较大虚拟网络的工作区会更快地用完 IP 地址（网络空间）。 例如，具有 /24 虚拟网络和 /26 子网的工作区一次最多可以有 64 个活动节点，而具有 /20 虚拟网络和 /22 子网的工作区最多可以容纳 1024 个节点。
  >
  > 子网将在你配置工作区时自动创建，并且你将有机会在配置期间为子网提供 CIDR 范围。

### <a name="configure-the-virtual-network"></a><a id="configure-the-virtual-network"> </a><a id="vnet-inject-advanced"> </a>配置虚拟网络

1. 在 Azure 门户中，选择“+ 创建资源”>“分析”>“Azure Databricks”或搜索 Azure Databricks，然后单击“创建”或“+ 添加”以启动“Azure Databricks 服务”对话框  。
2. 遵循[在自己的虚拟网络中部署 Azure Databricks 工作区](/azure-databricks/quickstart-create-databricks-workspace-vnet-injection)快速入门中所述的配置步骤。
3. 选择要使用的虚拟网络。

   > [!div class="mx-imgBorder"]
   > ![选择虚拟网络](../../../_static/images/vnet/vnet-injection-workspace.png)

4. 命名公共和专用子网，并在块中提供最高可达 /26 的 CIDR 范围：
   * 使用相关的[网络安全组规则](#nsg)创建一个公共子网，该规则允许与 Azure Databricks 控制平面进行通信。
   * 使用允许群集内部通信的相关网络安全组规则创建专用子网。
   * Azure Databricks 将拥有通过 `Microsoft.Databricks/workspaces` 服务更新两个子网的委托权限。 这些权限仅适用于 Azure Databricks 所需的网络安全组规则，而不适用于你添加的其他网络安全组规则或所有网络安全组中所包含的默认网络安全组规则。
5. 单击“创建”将 Azure Databricks 工作区部署到虚拟网络。

> [!NOTE]
>
> 当工作区部署失败时，仍然会在失败状态下创建工作区。 删除失败的工作区，并创建一个解决部署错误的新工作区。 删除失败的工作区时，托管资源组和任何成功部署的资源也将被删除。

## <a name="advanced-configuration-using-azure-resource-manager-templates"></a>使用 Azure 资源管理器模板的高级配置

如果想要更多地控制对虚拟网络的配置（例如，你想要使用现有子网、使用现有网络安全组，或者添加自己的安全规则），则可以使用以下 Azure 资源管理器 (ARM) 模板替代[基于门户 UI 的自动虚拟网络配置和工作区部署](#vnet-inject-portal)。

> [!IMPORTANT]
>
> 如果在 VNet 注入预览期间，在自己的 Azure 虚拟网络中部署了 Azure Databricks 工作区，则必须在 2020 年 3 月 31 日之前将预览工作区升级到正式发行版。 主要的升级任务是将公共和专用子网委托给 `Microsoft.Databricks/workspaces` 服务，使 Azure Databricks 能够创建[网络安全组规则](#nsg)。 如果需要添加或更新 Azure Databricks 托管 NSG 规则的范围，Azure Databricks 始终会提前通知。 请参阅[将“VNet 注入”预览版工作区升级到正式发行版](vnet-inject-upgrade.md#vnet-inject-upgrade)。
>
> 如果在预览期间使用 Azure 资源管理器模板将 Azure Databricks 工作区部署到自己的虚拟网络，并且想要继续使用 Azure 资源管理器模板来创建虚拟网络和部署工作区，则应使用以下升级的 Azure 资源管理器模板。 按照模板链接获取最新版本。
>
> 如果使用自定义 Azure 资源管理器模板或[用于 Databricks VNet 注入的工作区模板](https://azure.microsoft.com/resources/templates/101-databricks-workspace-with-vnet-injection)将工作区部署到现有虚拟网络，则必须创建公共和专用子网，将网络安全组附加到每个子网，并在部署工作区之前将子网委托给 `Microsoft.Databricks/workspaces` 服务 。 部署的每个工作区必须拥有一对单独的公共/专用子网。

### <a name="all-in-one-template"></a><a id="all-in-one-template"> </a><a id="arm-all-in-one"> </a>全功能模板

要在一个模板中创建虚拟网络和 Azure Databricks 工作区，请使用[适用于 Databricks VNet 注入工作区的全功能模板](https://azure.microsoft.com/resources/templates/101-databricks-all-in-one-template-for-vnet-injection)。

### <a name="virtual-network-template"></a><a id="arm-vnet-only"> </a><a id="virtual-network-template"> </a>虚拟网络模板

要创建具有适当公共和专用子网的虚拟网络，请使用[适用于 Databricks VNet 注入的虚拟网络模板](https://azure.microsoft.com/resources/templates/101-databricks-vnet-for-vnet-injection)。

### <a name="azure-databricks-workspace-template"></a><a id="arm-workspace"> </a><a id="azure-databricks-workspace-template"> </a>Azure Databricks 工作区模板

要将 Azure Databricks 工作区部署到现有的虚拟网络，请使用[适用于 Databricks VNet 注入的工作区模板](https://azure.microsoft.com/resources/templates/101-databricks-workspace-with-vnet-injection)。

> [!IMPORTANT]
>
> 在使用此 Azure 资源管理器模板部署工作区之前，虚拟网络的公共和专用子网必须附加网络安全组，并且必须将其委托给 `Microsoft.Databricks/workspaces` 服务。 可以使用[适用于 Databricks VNet 注入的虚拟网络模板](https://azure.microsoft.com/resources/templates/101-databricks-vnet-for-vnet-injection)来创建一个具有适当委托子网的虚拟网络。 如果使用现有的虚拟网络，并且尚未委托公共和专用子网，请参阅[添加或删除子网委托](/virtual-network/manage-subnet-delegation)或[将 VNet 注入预览工作区升级到正式发行版](vnet-inject-upgrade.md#vnet-inject-upgrade)。
>
> 部署的每个工作区必须拥有一对单独的公共/专用子网。

## <a name="network-security-group-rules"></a><a id="network-security-group-rules"> </a><a id="nsg"> </a>网络安全组规则

下表显示了 Azure Databricks 使用的最新网络安全组规则。 如果 Azure Databricks 需要添加规则或更改此列表中现有规则的范围，你将收到预先通知。 每当发生此类修改时，本文和各表都会更新。

### <a name="in-this-section"></a>本节内容：

* [Azure Databricks 如何管理网络安全组规则](#how-azure-databricks-manages-network-security-group-rules)
* [2020 年 1 月 13 日之后创建的工作区的网络安全组规则](#network-security-group-rules-for-workspaces-created-after-january-13-2020)
* [2020 年 1 月 13 日之前创建的工作区的网络安全组规则](#network-security-group-rules-for-workspaces-created-before-january-13-2020)

### <a name="how-azure-databricks-manages-network-security-group-rules"></a>Azure Databricks 如何管理网络安全组规则

下面各部分中列出的 NSG 规则表示 Azure Databricks 通过将虚拟网络的专用和公共子网委托给 `Microsoft.Databricks/workspaces` 服务，在 NSG 中自动预配和管理的规则。 你无权更新或删除这些 NSG 规则；子网委托会阻止任何更新或删除操作。 Azure Databricks 必须拥有这些规则才能确保 Microsoft 能够在虚拟网络中可靠地运行和支持 Azure Databricks 服务。 如果 Azure Databricks 需要添加规则或更改此列表中现有规则的范围，你将收到预先通知。

其中一些 NSG 规则将 VirtualNetwork 指定为源和目标。 在 Azure 中缺少子网级服务标记的情况下，这样做可以简化设计。 所有群集都在内部受到第二层网络策略的保护，这样群集 A 就无法连接到同一工作区中的群集 B。 如果工作区部署到同一客户管理的虚拟网络中的一对不同的子网中，这也适用于多个工作区。

> [!IMPORTANT]
>
> 如果工作区虚拟网络与另一个客户管理的网络对等互连，或者如果在其他子网中配置了非 Azure Databricks 资源，Databricks 建议你向附加到其他网络和子网的 NSG 添加拒绝入站规则，以阻止来自 Azure Databricks 群集的源流量。 你不需要为希望 Azure Databricks 群集连接到的资源添加此类规则。

### <a name="network-security-group-rules-for-workspaces-created-after-january-13-2020"></a>2020 年 1 月 13 日之后创建的工作区的网络安全组规则

下表仅适用于 2020 年 1 月 13 日之后创建的 Azure Databricks 工作区。 如果工作区是在 2020 年 1 月 13 日之前创建的，请参阅下表。

| 方向     | 协议     | 源                            | Source Port     | 目标                      | Dest Port     | 已使用          |
|---------------|--------------|-----------------------------------|-----------------|----------------------------------|---------------|---------------|
| 入站       | 任意          | VirtualNetwork                    | 任意             | VirtualNetwork                   | 任意           | 默认       |
| 入站       | TCP          | AzureDatabricks（服务标记）     | 任意             | VirtualNetwork                   | 22            | 公共 IP     |
| 入站       | TCP          | AzureDatabricks（服务标记）     | 任意             | VirtualNetwork                   | 5557          | 公共 IP     |
| 出站      | TCP          | VirtualNetwork                    | 任意             | AzureDatabricks（服务标记）    | 443           | 默认       |
| 出站      | TCP          | VirtualNetwork                    | 任意             | SQL                              | 3306          | 默认       |
| 出站      | TCP          | VirtualNetwork                    | 任意             | 存储                          | 443           | 默认       |
| 出站      | 任意          | VirtualNetwork                    | 任意             | VirtualNetwork                   | 任意           | 默认       |
| 出站      | TCP          | VirtualNetwork                    | 任意             | EventHub                         | 9093          | 默认       |

### <a name="network-security-group-rules-for-workspaces-created-before-january-13-2020"></a>2020 年 1 月 13 日之前创建的工作区的网络安全组规则

下表仅适用于 2020 年 1 月 13 日之前创建的 Azure Databricks 工作区。 如果工作区是在 2020 年 1 月 13 日当天或之后创建的，请参阅上表。

| 方向     | 协议     | 源              | Source Port     | 目标        | Dest Port     | 已使用          |
|---------------|--------------|---------------------|-----------------|--------------------|---------------|---------------|
| 入站       | 任意          | VirtualNetwork      | 任意             | VirtualNetwork     | 任意           | 默认       |
| 入站       | TCP          | ControlPlane IP     | 任意             | VirtualNetwork     | 22            | 公共 IP     |
| 入站       | TCP          | ControlPlane IP     | 任意             | VirtualNetwork     | 5557          | 公共 IP     |
| 出站      | TCP          | VirtualNetwork      | 任意             | Webapp IP          | 443           | 默认       |
| 出站      | TCP          | VirtualNetwork      | 任意             | SQL                | 3306          | 默认       |
| 出站      | TCP          | VirtualNetwork      | 任意             | 存储            | 443           | 默认       |
| 出站      | 任意          | VirtualNetwork      | 任意             | VirtualNetwork     | 任意           | 默认       |
| 出站      | TCP          | VirtualNetwork      | 任意             | EventHub           | 9093          | 默认       |

> [!IMPORTANT]
>
> Azure Databricks 是部署在全局 Azure 公有云基础结构上的 Microsoft Azure 第一方服务。 服务组件之间的所有通信（包括控制平面和客户数据平面中的公共 IP 之间的通信）都留在 Microsoft Azure 网络主干内进行。 另请参阅 [Microsoft 全球网络](/networking/microsoft-global-network)。

## <a name="troubleshooting"></a><a id="troubleshooting"> </a><a id="vnet-inject-troubleshoot"> </a>故障排除

### <a name="workspace-creation-errors"></a>工作区创建错误

子网<subnet ID>需要以下任意委托 [Microsoft.Databricks/workspaces] 来引用服务关联链接

可能的原因：创建工作区所在的 VNet 的专用子网和公共子网尚未委托给 `Microsoft.Databricks/workspaces` 服务。 每个子网必须附加并正确委托网络安全组。 有关详细信息，请参阅[虚拟网络要求](#vnet-inject-reqs)。

子网<subnet ID>已被工作区<workspace ID>占用

可能的原因：创建工作区所在的 VNet 的专用和公共子网已经被现有 Azure Databricks 工作区占用。 不能在单个子网中共享多个工作区。 部署的每个工作区必须拥有一对新的公共和专用子网。

### <a name="cluster-creation-errors"></a>群集创建错误

**实例不可访问：无法通过 SSH 访问资源。**

可能的原因：阻止了从控制平面到辅助角色的流量。 如果要部署到连接到本地网络的现有虚拟网络，请利用[将 Azure Databricks 工作区连接到本地网络](on-prem-network.md)中提供的信息检查安装。

**启动意外失败：设置群集时遇到意外错误。请重试，如果问题仍然存在，请联系 Azure Databricks。内部错误消息：`Timeout while placing node`。**

可能的原因：从辅助角色到 Azure 存储终结点的流量被阻止。 如果使用自定义 DNS 服务器，请同时检查虚拟网络中 DNS 服务器的状态。

**云提供程序启动失败：在设置群集时遇到云提供程序错误。请参阅 Azure Databricks 指南以获取详细信息。Azure 错误代码：`AuthorizationFailed/InvalidResourceReference.`**

可能的原因：虚拟网络或子网不存在。 请确保虚拟网络和子网存在。

**群集已终止。原因:Spark 启动失败：Spark 未能及时启动。此问题可能是由 Hive 元存储发生故障、Spark 配置无效或初始化脚本出现故障而导致的。请参阅 Spark 驱动程序日志以解决此问题，如果问题仍然存在，请联系 Databricks。内部错误消息：`Spark failed to start: Driver failed to start in time`。**

可能的原因：容器无法与托管实例或 DBFS 存储帐户通信。 解决方法是为 DBFS 存储帐户添加指向子网的自定义路由，并将下一个跃点设为 Internet。