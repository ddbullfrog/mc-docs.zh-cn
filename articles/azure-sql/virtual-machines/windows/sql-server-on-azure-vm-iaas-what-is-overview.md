---
title: Azure Windows 虚拟机上的 SQL Server 概述 | Microsoft Docs
description: 了解如何在云中的 Azure 虚拟机上运行完整版本的 SQL Server，而无需管理任何本地硬件。
services: virtual-machines-windows
documentationcenter: ''
author: WenJason
tags: azure-service-management
ms.assetid: c505089e-6bbf-4d14-af0e-dd39a1872767
ms.service: virtual-machines-sql
ms.topic: overview
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
origin.date: 11/27/2019
ms.date: 10/29/2020
ms.author: v-jay
ms.reviewer: jroth
ms.openlocfilehash: 7f41a319d8ed38f6061d5a5d713210818fa98714
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470488"
---
# <a name="what-is-sql-server-on-azure-virtual-machines-windows"></a>Azure 虚拟机 (Windows) 上的 SQL Server 是什么？
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
> * [Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md)

[Azure 虚拟机上的 SQL Server](https://www.azure.cn/home/features/virtual-machines/#virtual-machine-SQLserver) 允许你在云中使用完整版本的 SQL Server，而不需管理任何本地硬件。 使用标准预付费套餐时，SQL Server 虚拟机 (VM) 还可以简化许可成本。

<!--Correct on pay standard-in-advance offer-->

Azure 虚拟机在全球许多不同的[地理区域](https://azure.microsoft.com/regions/)运行， 并提供各种[虚拟机大小](../../../virtual-machines/windows/sizes.md)。 使用虚拟机映像库可以创建 SQL Server VM，而且版本和操作系统都很正确。 因此，虚拟机适用于许多不同的 SQL Server 工作负荷。

## <a name="automated-updates"></a>自动更新

Azure 虚拟机上的 SQL Server 可以使用[自动修补](automated-patching.md)来安排维护时段，以便自动安装重要的 Windows 和 SQL Server 更新。

## <a name="automated-backups"></a>自动备份

Azure 虚拟机上的 SQL Server 可以利用[自动备份](automated-backup.md)，定期创建数据库到 Blob 存储的备份。 也可以手动使用此技术。 有关详细信息，请参阅[使用 Azure 存储进行 SQL Server 备份和还原](azure-storage-sql-server-backup-restore-use.md)。

Azure 还为 Azure VM 中运行的 SQL Server 提供企业级备份解决方案。 作为完全托管的备份解决方案，它支持 AlwaysOn 可用性组、长期保留、时点恢复以及集中管理和监视。 有关详细信息，请参阅 [Azure VM 中 SQL Server 的 Azure 备份](/backup/backup-azure-sql-database)。
  

## <a name="high-availability"></a>高可用性

如果需要高可用性，请考虑配置 SQL Server 可用性组。 这涉及虚拟网络中 Azure 虚拟机上的 SQL Server 的多个实例。 可以手动配置高可用性解决方案，也可以在 Azure 门户中使用模板进行自动配置。 有关所有高可用性选项的概述，请参阅 [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](business-continuity-high-availability-disaster-recovery-hadr-overview.md)。

## <a name="performance"></a>性能

Azure 虚拟机提供的虚拟机大小取决于工作负荷需求。 SQL Server VM 还提供按性能要求优化的自动存储配置。 若要详细了解如何为 SQL Server VM 配置存储，请参阅 [SQL Server VM 的存储配置](storage-configuration.md)。 若要优化性能，请参阅 [Azure 虚拟机上 SQL Server 的性能最佳做法](performance-guidelines-best-practices.md)。

## <a name="get-started-with-sql-server-vms"></a>SQL Server VM 入门

若要开始，请选择一个 SQL Server 虚拟机映像，其中包含所需的版本和操作系统。 下面的各部分针对相关 SQL Server 虚拟机库映像提供了指向 Azure 门户的直接链接。

> [!TIP]
> 有关如何了解 SQL Server 映像定价的详细信息，请参阅 [Azure 虚拟机上的 SQL Server 定价指南](pricing-guidance.md)。 

<a name="payasyougo"></a>
<a name="payinadvance"></a>
### <a name="standard-pay-in-advance-offer"></a>标准预付费套餐
下表提供了标准预付费套餐 SQL Server 映像的矩阵。

| 版本 | 操作系统 | 版本 |
| --- | --- | --- |
| **SQL Server 2019** | Windows Server 2019 | [Enterprise](https://portal.azure.cn/#create/microsoftsqlserver.sql2019-ws2019enterprise)、[Standard](https://portal.azure.cn/#create/microsoftsqlserver.sql2019-ws2019standard)、[Web](https://portal.azure.cn/#create/microsoftsqlserver.sql2019-ws2019web)、[Developer](https://portal.azure.cn/#create/microsoftsqlserver.sql2019-ws2019sqldev) | 
| **SQL Server 2017** |Windows Server 2016 |[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2017EnterpriseWindowsServer2016)、[Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2017StandardonWindowsServer2016)、[Web](https://portal.azure.cn/#create/Microsoft.SQLServer2017WebonWindowsServer2016)、[Express](https://portal.azure.cn/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonWindowsServer2016)、[Developer](https://portal.azure.cn/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonWindowsServer2016) |
| **SQL Server 2016 SP2** |Windows Server 2016 |[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2016SP2EnterpriseWindowsServer2016)、[Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2016SP2StandardWindowsServer2016)、[Web](https://portal.azure.cn/#create/Microsoft.SQLServer2016SP2WebWindowsServer2016)、[Express](https://portal.azure.cn/#create/Microsoft.FreeLicenseSQLServer2016SP2ExpressWindowsServer2016)、[Developer](https://portal.azure.cn/#create/Microsoft.FreeLicenseSQLServer2016SP2DeveloperWindowsServer2016) |
| **SQL Server 2014 SP2** |Windows Server 2012 R2 |[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2014SP2EnterpriseWindowsServer2012R2)、[Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2014SP2StandardWindowsServer2012R2)、[Web](https://portal.azure.cn/#create/Microsoft.SQLServer2014SP2WebWindowsServer2012R2)、[Express](https://portal.azure.cn/#create/Microsoft.SQLServer2014SP2ExpressWindowsServer2012R2) |
| **SQL Server 2012 SP4** |Windows Server 2012 R2 |[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP4EnterpriseWindowsServer2012R2)、[Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP4StandardWindowsServer2012R2)、[Web](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP4WebWindowsServer2012R2)、[Express](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP4ExpressWindowsServer2012R2) |
| **SQL Server 2008 R2 SP3** |Windows Server 2008 R2|[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2008R2SP3EnterpriseWindowsServer2008R2)、[Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2008R2SP3StandardWindowsServer2008R2)、[Web](https://portal.azure.cn/#create/Microsoft.SQLServer2008R2SP3WebWindowsServer2008R2)、[Express](https://portal.azure.cn/#create/Microsoft.SQLServer2008R2SP3ExpressWindowsServer2008R2) |

若要查看可用的 Azure 虚拟机上的 SQL Server 映像，请参阅 [Azure 虚拟机上的 SQL Server 概述 (Linux)](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md)。

<!--Not Available on [How to change the licensing model for a SQL VM](virtual-machines-windows-sql-ahb.md)-->


### <a name="bring-your-own-license"></a><a id="BYOL"></a> 自带许可
也可以自带许可 (BYOL)。 在此方案中，你只需支付 VM 费用，SQL Server 许可不需要任何额外的费用。  自带许可证长时间会节省资金，因为可以持续使用生产型工作负荷。 有关使用此选项的要求，请参阅 [SQL Server Azure VM 定价指南](pricing-guidance.md#byol)。

若要自带许可证，可以转换现有的按用量付费的 SQL Server VM，也可以部署前缀为 {BYOL} 的映像。 

<!-- Not Available on [How to change the licensing model for a SQL VM](virtual-machines-windows-sql-ahb.md)-->

| 版本 | 操作系统 | 版本 |
| --- | --- | --- |
| **SQL Server 2019** | Windows Server 2019 | [Enterprise BYOL](https://portal.azure.cn/#create/microsoftsqlserver.sql2019-ws2019-byolenterprise)、[Standard BYOL](https://portal.azure.cn/#create/microsoftsqlserver.sql2019-ws2019-byolstandard)| 
| **SQL Server 2017** |Windows Server 2016 |[Enterprise BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2017EnterpriseWindowsServer2016)、[Standard BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2017StandardonWindowsServer2016) |
| **SQL Server 2016 SP2** |Windows Server 2016 |[Enterprise BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2016SP2EnterpriseWindowsServer2016)、[Standard BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2016SP2StandardWindowsServer2016) |
| **SQL Server 2014 SP2** |Windows Server 2012 R2 |[Enterprise BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2014SP2EnterpriseWindowsServer2012R2)、[Standard BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2014SP2StandardWindowsServer2012R2) |
| **SQL Server 2012 SP4** |Windows Server 2012 R2 |[Enterprise BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2012SP4EnterpriseWindowsServer2012R2)、[Standard BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2012SP4StandardWindowsServer2012R2) |

可以使用 PowerShell 部署 Azure 门户中不可用的较旧的 SQL Server 映像。 若要使用 PowerShell 查看所有可用映像，请使用以下命令：

  ```powershell
  Get-AzVMImageOffer -Location $Location -Publisher 'MicrosoftSQLServer'
  ```

有关使用 PowerShell 部署 SQL Server VM 的详细信息，请查看[如何使用 Azure PowerShell 预配 SQL Server 虚拟机](create-sql-vm-powershell.md)。


### <a name="connect-to-the-vm"></a>连接到 VM
创建 SQL Server VM 以后，即可从 SQL Server Management Studio (SSMS) 之类的应用程序或工具连接到该 VM。 有关说明，请参阅[连接到 Azure 上的 SQL Server 虚拟机](ways-to-connect-to-sql.md)。

### <a name="migrate-your-data"></a>迁移数据
如果已有数据库，会想要将该数据库移至新预配的 SQL Server VM。 有关迁移选项的列表和指导，请参阅[将数据库迁移到 Azure VM 上的 SQL Server](migrate-to-vm-from-sql-server.md)。

<!--Not Available on ## Create and manage Azure SQL resources with the Azure portal-->
<!--MOONCAKE: **Azure SQL** resource not exist on Azure China portal-->

## <a name="sql-server-vm-image-refresh-policy"></a><a id="lifecycle"></a> SQL Server VM 映像刷新策略
对于每种支持的操作系统和版本的组合，Azure 只保留一个虚拟机映像。 这意味着，随着时间的推移，映像会进行刷新，旧映像会被删除。 有关详细信息，请参阅[SQL Server VM 常见问题解答](frequently-asked-questions-faq.md#images)的“映像”部分。

## <a name="customer-experience-improvement-program-ceip"></a>客户体验改善计划 (CEIP)
客户体验改善计划 (CEIP) 默认情况下已启用。 它定期将报告发送给 Azure，以帮助改进 SQL Server。 CEIP 不要求管理任务，除非想在预配后禁用它。 可以通过远程桌面连接到 VM，以自定义或禁用 CEIP。 然后运行 **SQL Server 错误和使用情况报告** 实用工具。 请按照说明禁用报告功能。 有关数据收集的详细信息，请参阅 [SQL Server 隐私声明](https://docs.microsoft.com/sql/sql-server/sql-server-privacy)。

## <a name="related-products-and-services"></a>相关产品和服务
### <a name="windows-virtual-machines"></a>Windows 虚拟机
* [Azure 虚拟机概述](../../../virtual-machines/windows/overview.md)

### <a name="storage"></a>存储
* [Azure 存储简介](../../../storage/common/storage-introduction.md)

### <a name="networking"></a>网络
* [虚拟网络概述](../../../virtual-network/virtual-networks-overview.md)
* [Azure 中的 IP 地址](../../../virtual-network/public-ip-addresses.md)
* [在 Azure 门户中创建完全限定的域名](../../../virtual-machines/linux/portal-create-fqdn.md)

### <a name="sql"></a>SQL
* [SQL Server 文档](https://docs.microsoft.com/sql/index)

<!--Not Avaialble on * [Azure SQL Database comparison](../../azure-sql-iaas-vs-paas-what-is-overview.md)-->

## <a name="next-steps"></a>后续步骤

Azure 虚拟机上的 SQL Server 入门：

* [在 Azure 门户中创建 SQL Server VM](sql-vm-create-portal-quickstart.md)

获取有关 SQL Server VM 的常见问题的解答：

* [Azure 虚拟机中的 SQL Server 常见问题](frequently-asked-questions-faq.md)

<!--Not Available on * [Windows N-tier application on Azure with SQL Server](https://docs.microsoft.com/azure/architecture/reference-architectures/n-tier/n-tier-sql-server)-->
<!--Not Available on * [Run an N-tier application in multiple Azure regions for high availability](https://docs.microsoft.com/azure/architecture/reference-architectures/n-tier/multi-region-sql-server)-->

<!-- Update_Description: new article about sql server on azure vm iaas what is overview -->
<!--NEW.date: 07/06/2020-->