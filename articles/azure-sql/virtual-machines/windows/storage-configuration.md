---
title: SQL Server VM 的存储配置 | Microsoft Docs
description: 本主题介绍 Azure 在预配期间如何配置 SQL Server VM 的存储（Azure 资源管理器部署模型）。 此外，还说明了如何为现有的 SQL Server VM 配置存储。
services: virtual-machines-windows
documentationcenter: na
author: WenJason
tags: azure-resource-manager
ms.assetid: 169fc765-3269-48fa-83f1-9fe3e4e40947
ms.service: virtual-machines-sql
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
origin.date: 12/26/2019
ms.date: 10/29/2020
ms.author: v-jay
ms.openlocfilehash: 59ebe2955d3b4b20cef0cc942ce42dd7e1264711
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470040"
---
# <a name="storage-configuration-for-sql-server-vms"></a>SQL Server VM 的存储配置
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

在 Azure 中配置 SQL Server 虚拟机 (VM) 映像时，可以借助 Azure 门户自动完成存储配置。 这包括将存储附加到 VM、使该存储可供 SQL Server 访问，并对其进行配置以根据特定的性能要求优化。

本主题介绍 Azure 如何在预配期间针对 SQL Server VM 以及针对现有的 VM 配置存储。 此配置基于运行 SQL Server 的 Azure VM 的[性能最佳做法](performance-guidelines-best-practices.md)。

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="prerequisites"></a>先决条件

若要使用自动存储配置设置，虚拟机需有以下特征：

* 已使用 [SQL Server 库映像](sql-server-on-azure-vm-iaas-what-is-overview.md#payasyougo)预配。
* 使用 [Resource Manager 部署模型](../../../azure-resource-manager/management/deployment-models.md)。
* 使用[高级 SSD](../../../virtual-machines/disks-types.md)。

## <a name="new-vms"></a>新的 VM

以下部分介绍了如何为新的 SQL Server 虚拟机配置存储。

### <a name="azure-portal"></a>Azure 门户

使用 SQL Server 库映像预配 Azure VM 时，可以选择自动为新的 VM 配置存储。 可以指定存储大小、性能限制和工作负荷类型。 以下屏幕截图显示了在预配 SQL VM 期间使用的“存储配置”边栏选项卡。

![预配期间的 SQL Server VM 存储配置](./media/storage-configuration/sql-vm-storage-configuration-provisioning.png)

在 **存储优化** 下选择要为其部署 SQL Server 的工作负荷类型。 使用“常规”优化选项，默认情况下，你将拥有一个最大 IOPS 为 5000 的数据磁盘，并且你将使用此同一驱动器放置数据、事务日志和 TempDB 存储。 你还可以根据业务选择“事务处理”(OLTP) 或“数据仓库”。

<!--MOONCAKE: CUSTOMIZATION Remove the descriptions about three kind of VM performance-->

![预配期间的 SQL Server VM 存储配置](./media/storage-configuration/sql-vm-storage-configuration.png)

<!--Remove the customized on configuration-->
<!--Not exists on Azure China Cloud-->

根据所做的选择，Azure 会在创建 VM 后执行以下存储配置任务：

* 创建高级 SSD 盘并将其连接到虚拟机。
* 配置 SQL Server 可访问的数据磁盘。
* 根据指定的大小和性能（IOPS 和吞吐量）要求，在存储池中配置数据磁盘。
* 将存储池与虚拟机上的新驱动器相关联。
* 根据指定的工作负荷类型（“数据仓库”、“事务处理”或“常规”）优化此新驱动器。

有关 Azure 如何配置存储设置的详细信息，请参阅 [存储配置部分](#storage-configuration)。 有关如何在 Azure 门户中创建 SQL Server VM 的完整演练，请参阅[预配教程](../../../azure-sql/virtual-machines/windows/create-sql-vm-portal.md)。

### <a name="resource-manager-templates"></a>Resource Manager 模板

如果使用以下 Resource Manager 模板，则会默认附加两个不带存储池配置的高级数据磁盘。 但是，可以自定义这些模板，更改附加到虚拟机的高级数据磁盘的数目。

>[!NOTE]
> 必须修改从 GitHub 存储库“azure-quickstart-templates”下载或参考的模板，以适应 Azure 中国云环境。 例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；必要时更改某些不受支持的 VM 映像、VM 大小、SKU 以及资源提供程序的 API 版本。


* [使用自动备份创建 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-sql-full-autobackup)
* [使用自动修补创建 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-sql-full-autopatching)
* [使用 AKV 集成创建 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-sql-full-keyvault)

### <a name="quickstart-template"></a>快速入门模板

可以使用以下快速入门模板通过存储优化来部署 SQL Server VM。 

* [通过存储优化创建 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-vm-new-storage/)
* [创建使用 UltraSSD 的 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-vm-new-storage-ultrassd)

## <a name="existing-vms"></a>现有 VM

<!--MOONCAKE: CUSTOMIZE on 10/10/2019-->
<!--Not Available on [!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]-->

对于现有的 SQL Server VM，可以在 Azure 门户中修改某些存储设置。 选择 VM，转到“设置”区域，并选择“SQL Server 配置”。 

<!--Not Avaialble on The SQL Server Configuration blade shows the current storage usage of your VM. All drives that exist on your VM are displayed in this chart. For each drive, the storage space displays in four sections:-->
<!--Not Available on [SQL virtual machines resource](virtual-machines-windows-sql-manage-portal.md#access-sql-virtual-machine-resource)-->
<!--MOONCAKE: CUSTOMIZE on 10/10/2019-->

![为现有 SQL Server VM 配置存储](./media/storage-configuration/sql-vm-storage-configuration-existing.png)

显示的配置选项会有所不同，这取决于以前是否用过此功能。 首次使用时，可以指定新驱动器的存储要求。 如果以前曾经使用此功能创建了驱动器，可以选择扩展该驱动器的存储。

### <a name="use-for-the-first-time"></a>首次使用

如果首次使用此功能，可以指定新驱动器的存储大小和性能限制。 这种体验与预配类似。 主要差别在于无法指定工作负荷类型。 此限制可防止中断虚拟机上任何现有的 SQL Server 配置。

Azure 根据规范创建新驱动器。 在此方案中，Azure 将执行以下存储配置任务：

* 创建高级存储数据磁盘并将其连接到虚拟机。
* 配置 SQL Server 可访问的数据磁盘。
* 根据指定的大小和性能（IOPS 和吞吐量）要求，在存储池中配置数据磁盘。
* 将存储池与虚拟机上的新驱动器相关联。

有关 Azure 如何配置存储设置的详细信息，请参阅 [存储配置部分](#storage-configuration)。

### <a name="add-a-new-drive"></a>添加新驱动器

如果已在 SQL Server VM 上配置存储，则展开存储会显示两个新选项。 第一个选项是添加新驱动器以提升 VM 的性能级别。

但是，添加驱动器后，必须执行一些附加的手动配置才能提升性能。

### <a name="extend-the-drive"></a>扩展驱动器

扩展存储的另一个选项是扩展现有驱动器。 此选项会增加驱动器的可用存储，但不提升性能。 对于存储池，在创建存储池后无法更改列数。 列数决定了可跨数据磁盘条带化的并行写入数。 因此，添加的任何数据磁盘均无法提升性能。 它们只能为写入的数据提供更多的存储空间。 这种限制也意味着，在扩展驱动器时，列数决定了可以添加的数据磁盘的最小数目。 因此，如果创建的存储池包含四个数据磁盘，则列数也是四个。 每次扩展存储时，必须至少添加四个数据磁盘。

![扩展 SQL VM 的驱动器](./media/storage-configuration/sql-vm-storage-extend-drive.png)

<!--MOONCAKE: CUSTOMIZE on 10/10/2019-->

## <a name="storage-configuration"></a>存储配置

本部分提供有关在 Azure 门户中预配或配置 SQL Server VM 期间，Azure 自动执行的存储配置更改的参考信息。

* Azure 通过从 VM 中选择的存储配置存储池。 本主题的下一部分提供了有关存储池配置的详细信息。
* 自动存储配置始终使用[高级 SSD](../../../virtual-machines/disks-types.md) P30 数据磁盘。 因此，所选 TB 数目与附加到 VM 的数据磁盘数目之间存在 1:1 映射。

有关价格信息，请参阅“磁盘存储”选项卡上的[存储定价](https://www.azure.cn/pricing/details/storage/)页。

### <a name="creation-of-the-storage-pool"></a>创建存储池

Azure 使用以下设置在 SQL Server VM 上创建存储池。

| 设置 | Value |
| --- | --- |
| 条带大小 |256 KB（数据仓库）；64 KB（事务） |
| 磁盘大小 |每个磁盘 1 TB |
| 缓存 |读取 |
| 分配大小 |64 KB NTFS 分配单元大小 |
| 恢复 | 简单恢复（不可复原） |
| 列数 |数据磁盘数最多 8 个<sup>1</sup> |


<sup>1</sup> 创建存储池后，无法更改存储池中的列数。


## <a name="workload-optimization-settings"></a>工作负荷优化设置

下表描述了三个可用的工作负荷类型选项及其对应的优化：

| 工作负荷类型 | 说明 | 优化 |
| --- | --- | --- |
| **常规** |支持大多数工作负荷的默认设置 |无 |
| **事务处理** |针对传统数据库 OLTP 工作负荷优化存储 |跟踪标志 1117<br/>跟踪标志 1118 |
| **数据仓库** |针对分析和报告工作负荷优化存储 |跟踪标志 610<br/>跟踪标志 1117 |

> [!NOTE]
> 只有在预配 SQL Server 虚拟机时才能指定工作负载类型，方法是在存储配置步骤中进行选择。

## <a name="next-steps"></a>后续步骤

有关其他与在 Azure VM 中运行 SQL Server 相关的主题，请参阅 [SQL Server on Azure Virtual Machines](sql-server-on-azure-vm-iaas-what-is-overview.md)（Azure 虚拟机上的 SQL Server）。

<!-- Update_Description: new article about storage configuration -->
<!--NEW.date: 07/06/2020-->