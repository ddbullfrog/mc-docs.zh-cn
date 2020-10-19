---
title: 恢复服务保管库概述
description: 恢复服务保管库概述。
author: Johnnytechn
origin.date: 08/10/2018
ms.topic: conceptual
ms.date: 09/28/2020
ms.author: v-johya
ms.openlocfilehash: b46866d0f6ffb4f1d50e6b340601d19690c3c567
ms.sourcegitcommit: 80567f1c67f6bdbd8a20adeebf6e2569d7741923
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91871174"
---
# <a name="recovery-services-vaults-overview"></a>恢复服务保管库概述

本文介绍恢复服务保管库的功能。 恢复服务保管库是 Azure 中用于存储数据的存储实体。 数据通常是虚拟机 (VM)、工作负荷、服务器或工作站的数据或配置信息的副本。 可以使用恢复服务保管库为各种 Azure 服务（例如 IaaS VM（Linux 或 Windows））和 Azure SQL 数据库存储备份数据。 恢复服务保管库支持 System Center DPM、Windows Server、Azure 备份服务器等。 使用恢复服务保管库可以方便地组织备份数据，并将管理开销降至最低。 恢复服务保管库基于 Azure 的 Azure 资源管理器模型，该模型提供如下功能：

- **有助于确保备份数据安全的增强功能**：使用恢复服务保管库时，Azure 备份提供用于保护云备份的安全功能。 这些安全功能确保可以保护备份并安全地恢复数据，即使生产服务器和备份服务器受到危害。 [了解详细信息](backup-azure-security-feature.md)

- **针对混合 IT 环境进行集中监视**：使用恢复服务保管库时，可以通过中心门户监视 [Azure IaaS VM](backup-azure-manage-vms.md) 和[本地资产](backup-azure-manage-windows-server.md#manage-backup-items)。 [了解详细信息](backup-azure-monitoring-built-in-monitor.md)

- **基于角色的访问控制 (RBAC)** ：RBAC 在 Azure 中提供精细的访问管理控制。 [Azure 提供各种内置角色](../role-based-access-control/built-in-roles.md)，而 Azure 备份具有三个[用于管理恢复点的内置角色](backup-rbac-rs-vault.md)。 恢复服务保管库与 RBAC 兼容，后者会限制对已定义用户角色集的备份和还原访问权限。 [了解详细信息](backup-rbac-rs-vault.md)

- 软删除：在使用软删除的情况下，即使恶意行动者删除了备份（或用户意外删除了备份数据），备份数据也仍会保留 14 天，因此可以恢复该备份项，而不会丢失数据。 以“软删除”状态将备份数据额外保留 14 天不会向你收取任何费用。 [了解详细信息](backup-azure-security-feature-cloud.md)。

<!--Not available in MC: **Cross Region Restore**-->
## <a name="storage-settings-in-the-recovery-services-vault"></a>恢复服务保管库中的存储设置

恢复服务保管库是用于存储在不同时间创建的备份和恢复点的实体。 恢复服务保管库还包含与受保护虚拟机关联的备份策略。

- Azure 备份会自动处理保管库的存储。 查看如何[更改存储设置](./backup-create-rs-vault.md#set-storage-redundancy)。

- 若要详细了解存储冗余，请参阅有关[异地](../storage/common/storage-redundancy.md)冗余和[本地](../storage/common/storage-redundancy.md#locally-redundant-storage)冗余的这些文章。

## <a name="encryption-settings-in-the-recovery-services-vault"></a>恢复服务保管库中的加密设置

本部分介绍可用于加密恢复服务保管库中存储的备份数据的选项。

### <a name="encryption-of-backup-data-using-platform-managed-keys"></a>使用平台托管的密钥加密备份数据

默认情况下，所有数据将使用平台托管的密钥进行加密。 无需从你的终端执行任何明确操作即可实现此加密。 这种加密适用于要备份到恢复服务保管库的所有工作负荷。

<!--Not available in MC: ### Encryption of backup data using customer-managed keys-->
## <a name="azure-advisor"></a>Azure 顾问

[Azure 顾问](../advisor/index.yml)是个性化的云顾问，可帮助优化 Azure 的使用。 它会分析 Azure 的使用情况，并提供及时的建议来帮助优化和保护部署。 它提供四个类别的建议：高可用性、安全性、性能和成本。

Azure 顾问为未备份的 VM 提供每小时[建议](../advisor/advisor-high-availability-recommendations.md#protect-your-virtual-machine-data-from-accidental-deletion)，因此，你永远不会错过备份重要的 VM。 你还可以通过推迟建议来控制建议。  可选择建议，然后通过指定保管库（将在其中存储备份）和备份策略（备份计划和备份副本保留期）来在 VM 上启用内联备份。

![Azure 顾问](./media/backup-azure-recovery-services-vault-overview/azure-advisor.png)

## <a name="additional-resources"></a>其他资源

- [支持和不支持保管库的方案](backup-support-matrix.md#vault-support)
- [保管库常见问题解答](backup-azure-backup-faq.md)

## <a name="next-steps"></a>后续步骤

使用以下文章了解相关操作：

- [备份 IaaS VM](backup-azure-arm-vms-prepare.md)
- [备份 Azure 备份服务器](backup-azure-microsoft-azure-backup.md)
- [备份 Windows Server](backup-windows-with-mars-agent.md)

<!-- Update_Description: wording update -->
