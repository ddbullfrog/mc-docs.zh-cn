---
title: 关于 Azure 虚拟机还原过程
description: 了解 Azure 备份服务如何还原 Azure 虚拟机
ms.topic: conceptual
ms.date: 09/28/2020
author: Johnnytechn
ms.author: v-johya
ms.openlocfilehash: 9b2a755df67fb6ac5bdd20958afe1728f1c5c8b4
ms.sourcegitcommit: 80567f1c67f6bdbd8a20adeebf6e2569d7741923
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91871201"
---
# <a name="about-azure-vm-restore"></a>关于 Azure VM 还原

本文介绍了 [Azure 备份服务](./backup-overview.md)如何还原 Azure 虚拟机 (VM)。 有许多还原选项可用。 我们将讨论它们支持的各种方案。

## <a name="concepts"></a>概念

- **恢复点**（也称为**还原点**）：恢复点是要备份的原始数据的副本。

- **层（快照与保管库）** ：Azure VM 备份分两个阶段进行：

  - 在第 1 阶段中，创建的快照与磁盘一起存储。 这称为**快照层**。 与从保管库还原相比，快照层还原的速度更快，因为它们没有在触发还原之前等待快照复制到保管库的时间。 因此，从快照层进行还原也称为[即时还原](./backup-instant-restore-capability.md)。
  - 第 2 阶段将传输快照并将其存储在由 Azure 备份服务管理的保管库中。 这称为**保管库层**。

- **原始位置恢复 (OLR)** ：执行从还原点到源 Azure VM（已在其中创建了备份）的恢复，将其替换为存储在恢复点中的状态。 这将替换源 VM 的 OS 磁盘和数据磁盘。

- **备用位置恢复 (ALR)** ：执行从恢复点到服务器（不是在其中创建了备份的源服务器）的恢复。

- **项级还原 (ILR)：** 从恢复点还原 VM 内的各个文件或文件夹

- **可用性（复制类型）** ：Azure 备份提供了两种类型的复制来保持存储/数据的高可用性：
  - [本地冗余存储 (LRS)](../storage/common/storage-redundancy.md#locally-redundant-storage) 将数据中心的存储缩放单元中的数据复制三次（创建三个数据副本）。 数据的所有副本存在于同一区域。 LRS 是一个低成本选项，可在本地硬件故障时保护数据。
  - [异地冗余存储 (GRS)](../storage/common/storage-redundancy.md#geo-redundant-storage) 是默认的和推荐的复制选项。 GRS 将数据复制到离源数据主位置数英里之外的次要区域中。 GRS 的成本比 LRS 的高，但 GRS 提供更高的数据持久度，即使出现区域性服务中断也是如此。

- **跨区域还原 (CRR)** ：作为[还原选项](./backup-azure-arm-restore-vms.md#restore-options)之一，跨区域还原 (CRR) 允许你在某个次要区域（[Azure 配对区域](../best-practices-availability-paired-regions.md#what-are-paired-regions)）中还原 Azure VM。

## <a name="restore-scenarios"></a>还原方案

![还原方案 ](./media/about-azure-vm-restore/recovery-scenarios.png)

| **方案**                                                 | **执行什么操作**                                             | **使用时机**                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [通过还原来创建新虚拟机](./backup-azure-arm-restore-vms.md) | 将整个 VM 还原到 OLR（如果源 VM 仍然存在）或 ALR | <li> 如果源 VM 丢失或损坏，则可以还原整个 VM  <li> 你可以创建 VM 的副本  <li> 你可以针对审核或合规性执行还原演练  <li> 此选项不适用于根据市场映像创建的 Azure VM（原因在于，这些映像可能由于许可证过期而无法使用）。 |
| [还原 VM 的磁盘](./backup-azure-arm-restore-vms.md#restore-disks) | 还原附加到 VM 的磁盘                             |  所有磁盘：此选项将创建模板并还原磁盘。 你可以使用特殊配置（例如，可用性集）编辑此模板以满足你的要求，然后使用模板并还原磁盘来重新创建 VM。 |
| [还原 VM 内的特定文件](./backup-azure-restore-files-from-vm.md) | 选择还原点，浏览，选择文件并将它们还原到与备份的 VM 相同的（或兼容的）OS。 |  如果你知道要还原的具体文件，请使用此选项而不是还原整个 VM。 |
| [还原加密的 VM](./backup-azure-vms-encryption.md) | 在门户中，还原磁盘，然后使用 PowerShell 创建 VM | <li> [带 Azure Active Directory 的加密 VM](../virtual-machines/windows/disk-encryption-windows-aad.md)  <li> [不带 Azure AD 的加密 VM](../virtual-machines/windows/disk-encryption-windows.md) <li> [带 Azure AD 的加密 VM 迁移到不带 Azure AD 的加密 VM ](../virtual-machines/windows/disk-encryption-faq.md#can-i-migrate-vms-that-were-encrypted-with-an-azure-ad-app-to-encryption-without-an-azure-ad-app) |

## <a name="next-steps"></a>后续步骤

- [有关 VM 还原的常见问题](./backup-azure-vm-backup-faq.md#restore)
- [支持的还原方法](./backup-support-matrix-iaas.md#supported-restore-methods)
- [排查还原问题](./backup-azure-vms-troubleshoot.md#restore)

