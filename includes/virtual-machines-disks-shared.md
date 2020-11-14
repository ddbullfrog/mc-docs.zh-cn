---
title: include 文件
description: include 文件
services: virtual-machines
ms.service: virtual-machines
ms.topic: include
origin.date: 07/14/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: yes|no
ms.testdate: 11/09/2020null
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: e3d64ea39e027ff53ceed41aebb3a637ee49ac74
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94329222"
---
<!--Verified successfully for PG notification-->
Azure 共享磁盘是 Azure 托管磁盘的一项新功能，可同时将托管磁盘附加到多个虚拟机 (VM)。 通过将托管磁盘附加到多个 VM，可以向 Azure 部署新的群集应用程序或迁移现有的群集应用程序。

## <a name="how-it-works"></a>工作原理

群集中的 VM 可根据群集应用程序使用 [SCSI 永久预留](https://www.t10.org/members/w_spc3.htm) (SCSI PR) 选择的预留来读取或写入附加的磁盘。 SCSI PR 是一种行业标准，可供本地存储区域网络 (SAN) 上运行的应用程序利用。 在托管磁盘上启用 SCSI PR，可以将这些应用程序按原样迁移到 Azure。

共享托管磁盘提供了可从多个 VM 进行访问的共享块存储，这些存储作为逻辑单元号 (LUN) 公开。 然后，会将 LUN 从目标（磁盘）提供给发起程序 (VM)。 这些 LUN 看起来像直接附加存储 (DAS) 或 VM 的本地驱动器。

共享托管磁盘本身并不提供可以使用 SMB/NFS 访问的完全托管的文件系统。 需要使用群集管理器（如 Windows Server 故障转移群集 (WSFC) 或 Pacemaker）来处理群集节点通信和写入锁定。

## <a name="limitations"></a>限制

[!INCLUDE [virtual-machines-disks-shared-limitations](virtual-machines-disks-shared-limitations.md)]

### <a name="operating-system-requirements"></a>操作系统要求

共享磁盘支持多个操作系统。 有关支持的操作系统，请参阅 [Windows](#windows) 或 [Linux](#linux) 部分。

## <a name="disk-sizes"></a>磁盘大小

[!INCLUDE [virtual-machines-disks-shared-sizes](virtual-machines-disks-shared-sizes.md)]

## <a name="sample-workloads"></a>示例工作负载

### <a name="windows"></a><a name="windows"></a>Windows

Windows Server 2008 及更高版本支持 Azure 共享磁盘。 大多数基于 Windows 的群集构建于 WSFC 上，它处理群集节点通信的所有核心基础结构，使应用程序能够利用并行访问模式。 WSFC 根据 Windows Server 的版本启用 CSV 和非 CSV 的选项。 有关详细信息，请参阅[创建故障转移群集](https://docs.microsoft.com/windows-server/failover-clustering/create-failover-cluster)。

WSFC 上运行的热门应用程序包括：

<!--Not Available on - [Create an FCI with Azure shared disks (SQL Server on Azure VMs)](../articles/azure-sql/virtual-machines/windows/failover-cluster-instance-azure-shared-disks-manually-configure.md)-->

- 横向扩展文件服务器 (SoFS) [模板] (https://aka.ms/azure-shared-disk-sofs-template)
- SAP ASCS/SCS [模板] (https://aka.ms/azure-shared-disk-sapacs-template)
- 常规用途的文件服务器（IW 工作负载）
- 远程桌面服务器用户配置文件磁盘 (RDS UPD)

### <a name="linux"></a><a name="linux"></a>Linux

支持 Azure 共享磁盘的项如下所示：
- [SUSE SLE for SAP 和 SUSE SLE HA 15 SP1 及更高版本](https://documentation.suse.com/sle-ha/15-SP1/single-html/SLE-HA-guide/index.html)
- [Ubuntu 18.04 及更高版本](https://discourse.ubuntu.com/t/ubuntu-high-availability-corosync-pacemaker-shared-disk-environments/14874)
- [任何 RHEL 8 版本上的 RHEL 开发人员预览版](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_high_availability_clusters/index)
- [Oracle Enterprise Linux](https://docs.oracle.com/en/operating-systems/oracle-linux/8/availability/hacluster-1.html)

Linux 群集可以利用群集管理器，例如 [Pacemaker](https://wiki.clusterlabs.org/wiki/Pacemaker)。 Pacemaker 基于 [Corosync](http://corosync.github.io/corosync/) 构建，可为部署在高可用环境中的应用程序启用群集通信。 一些常见的群集文件系统包括 [ocfs2](https://oss.oracle.com/projects/ocfs2/) 和 [gfs2](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/global_file_system_2/ch-overview-gfs2)。 可使用基于 SCSI 永久预留 (SCSI PR) 和/或 STONITH 块设备 (SBD) 的群集模型对磁盘进行仲裁访问。 使用 SCSI PR 时，可使用 [fence_scsi](http://manpages.ubuntu.com/manpages/eoan/man8/fence_scsi.8.html) 和 [sg_persist](https://linux.die.net/man/8/sg_persist) 之类的实用程序来处理预留和注册。

## <a name="persistent-reservation-flow"></a>永久预留流

下图演示了一个示例 2 节点群集数据库应用程序，该应用程序利用 SCSI PR 启用从一个节点到另一个节点的故障转移。

:::image type="content" source="media/virtual-machines-disks-shared-disks/shared-disk-updated-two-node-cluster-diagram.png" alt-text="双节点群集。群集上运行的应用程序正在处理对磁盘的访问":::

工作流如下所述：

1. 在 Azure VM1 和 VM2 上运行的群集应用程序均注册了其读取或写入磁盘的意图。
1. 然后，VM1 上的应用程序实例将使用独占预留以写入磁盘。
1. 已在 Azure 磁盘上强制执行此预留，并且数据库现在可以独占方式写入磁盘。 从 VM2 上的应用程序实例进行的任何写入都不会成功。
1. 如果 VM1 上的应用程序实例关闭，则 VM2 上的实例现在可以启动数据库故障转移和磁盘接管。
1. 现在，已在 Azure 磁盘上强制执行此预留，并且该磁盘将不再接受来自 VM1 的写入。 它将只接受来自 VM2 的写入。
1. 群集应用程序可以完成数据库故障转移并处理来自 VM2 的请求。

下图说明了另一个常见的群集工作负载，该工作负载由多个节点组成，这些节点从磁盘读取数据以运行并行进程，例如机器学习模型训练。

:::image type="content" source="media/virtual-machines-disks-shared-disks/shared-disk-updated-machine-learning-trainer-model.png" alt-text="四节点 VM 群集，每个节点注册写入意图，应用程序进行独占预留以正确处理写入结果":::

工作流如下所述：

1. 所有 VM 上运行的群集应用程序均注册了其读取或写入磁盘的意图。
1. 然后，VM1 上的应用程序实例将使用独占预留以写入磁盘，同时开放其他 VM 的磁盘读取。
1. 将在 Azure 磁盘上强制执行此预留。
1. 群集中的所有节点现在都可以从磁盘中读取。 只有一个节点代表群集中的所有节点将结果写回磁盘。

<!--Not Available on ### Ultra disks reservation flow-->

## <a name="performance-throttles"></a>性能限制

### <a name="premium-ssd-performance-throttles"></a>高级 SSD 性能限制

使用高级 SSD 时，磁盘 IOPS 和吞吐量是固定的；例如，P30 的 IOPS 是 5000。 无论磁盘是跨 2 个 VM 还是 5 个 VM 进行共享，都保留此值。 可从单个 VM 达到磁盘限额，也可跨两个或多个 VM 分配这些限额。 

<!--Not Available on ### Ultra disk performance throttles-->

<!--Not Available on FEATURE Ultra disk-->

<!--Not Available on #### Examples-->
<!--Not Available on ##### Two nodes cluster using cluster shared volumes-->

<!--Not Available on ##### Two node cluster without cluster share volumes-->
<!--Not Available on ##### Four node Linux cluster-->
<!--Not Available on #### Ultra pricing-->
<!-- Update_Description: new article about virtual machines disks shared -->
<!--NEW.date: 11/09/2020-->