---
title: 创建和配置恢复服务保管库
description: 本文介绍如何创建和配置用于存储备份和恢复点的恢复服务保管库。
author: Johnnytechn
ms.topic: conceptual
origin.date: 08/30/2019
ms.date: 09/22/2020
ms.author: v-johya
ms.openlocfilehash: 296c7d7ba127b522834a0380a8532e62065db92f
ms.sourcegitcommit: cdb7228e404809c930b7709bcff44b89d63304ec
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/28/2020
ms.locfileid: "91402635"
---
# <a name="create-and-configure-a-recovery-services-vault"></a>创建和配置恢复服务保管库

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="set-storage-redundancy"></a>设置存储冗余

Azure 备份会自动处理保管库的存储。 需要指定如何复制该存储。

> [!NOTE]
> 在保管库中配置备份之前，必须更改恢复服务保管库的“存储复制类型”（本地冗余/异地冗余）。 配置备份后，将禁用修改选项。
>
>- 如果尚未配置备份，请[遵循这些步骤](#set-storage-redundancy)检查和修改设置。
>- 如果已配置备份，并且必须从 GRS 迁移到 LRS，请[查看这些解决方法](#how-to-change-from-grs-to-lrs-after-configuring-backup)。

1. 在“恢复服务保管库”窗格中，选择新保管库。 在“设置”部分下，选择“属性”。 
1. 在“属性”中的“备份配置”下，选择“更新”。  

1. 选择存储复制类型，然后选择“保存”。

     ![设置新保管库的存储配置](./media/backup-try-azure-backup-in-10-mins/recovery-services-vault-backup-configuration.png)

   - 如果使用 Azure 作为主要备份存储终结点，则我们建议继续使用默认的“异地冗余”设置。
   - 如果不使用 Azure 作为主要的备份存储终结点，则请选择“本地冗余”，减少 Azure 存储费用。
   - 详细了解[异地冗余](../storage/common/storage-redundancy.md)和[本地冗余](../storage/common/storage-redundancy.md)。

<!--Not available in MC: Azure file share-->

<!--Not available in MC: ## Set encryption settings-->
## <a name="modifying-default-settings"></a>修改默认设置

在保管库中配置备份之前，我们强烈建议检查“存储复制类型”和“安全设置”的默认设置。 

- “存储复制类型”默认设置为“异地冗余”(GRS) 。 配置备份后，将禁用修改选项。
  - 如果尚未配置备份，请[遵循这些步骤](#set-storage-redundancy)检查和修改设置。
  - 如果已配置备份，并且必须从 GRS 迁移到 LRS，请[查看这些解决方法](#how-to-change-from-grs-to-lrs-after-configuring-backup)。

- “软删除”对新建的保管库默认为“已启用”，旨在防止意外或恶意删除备份数据。  [遵循这些步骤](./backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete)检查和修改设置。

### <a name="how-to-change-from-grs-to-lrs-after-configuring-backup"></a>配置备份后如何从 GRS 更改为 LRS

在决定从 GRS 改为本地冗余存储 (LRS) 之前，请先根据你的应用场景在较低成本和较高数据持续性之间权衡。 如果必须从 GRS 改用 LRS，你有两种选择。 它们取决于保留备份数据的业务要求：

- [无需保留以前备份的数据](#dont-need-to-preserve-previous-backed-up-data)
- [必须保留以前备份的数据](#must-preserve-previous-backed-up-data)

#### <a name="dont-need-to-preserve-previous-backed-up-data"></a>无需保留以前备份的数据

若要在新的 LRS 保管库中保护工作负载，需在 GRS 保管库中删除当前保护和数据，并重新配置备份。

>[!WARNING]
>以下操作是破坏性的，无法撤消。 与受保护服务器关联的所有备份数据和备份项将被永久删除。 请谨慎操作。

停止并删除对 GRS 保管库的当前保护：

1. 在 GRS 保管库属性中禁用软删除。 按照[这些步骤](backup-azure-security-feature-cloud.md#disabling-soft-delete-using-azure-portal)禁用软删除。

1. 停止保护并删除现有 GRS 保管库中的备份。 在保管库仪表板菜单中，选择“备份项”。 此处列出的需要移动到 LRS 保管库的项必须连同其备份数据一起删除。 请参阅如何[删除云中受保护的项](backup-azure-delete-vault.md#delete-protected-items-in-the-cloud)以及[删除本地受保护的项](backup-azure-delete-vault.md#delete-protected-items-on-premises)。

1. 如果计划迁移 SQL 服务器或 SAP HANA 服务器，则还需要取消注册它们。 在保管库仪表板菜单中，选择“备份基础结构”。 请参阅如何[取消注册 SQL 服务器](manage-monitor-sql-database-backup.md#unregister-a-sql-server-instance)以及[取消注册 SAP HANA 实例](sap-hana-db-manage.md#unregister-an-sap-hana-instance)。

1. 将其从 GRS 保管库中删除后，请继续在新的 LRS 保管库中配置工作负载的备份。

#### <a name="must-preserve-previous-backed-up-data"></a>必须保留以前备份的数据

如果需要将当前受保护的数据保留在 GRS 保管库中，并在新的 LRS 保管库中继续保护，则某些工作负载的选项受到限制：

- 对于 MARS，可以[停止保护并保留数据](backup-azure-manage-mars.md#stop-protecting-files-and-folder-backup)并在新的 LRS 保管库中注册代理。

  - Azure 备份服务将继续保留 GRS 保管库的所有现有恢复点。
  - 需要付费才能将恢复点保留在 GRS 保管库中。
  - 只能还原 GRS 保管库中尚未过期的恢复点的备份数据。
  - 需要在 LRS 保管库中创建数据的新初始副本。

- 对于 Azure VM，可以对 GRS 保管库中的 VM [停止保护并保留数据](backup-azure-manage-vms.md#stop-protecting-a-vm)，将该 VM 移到其他资源组，然后在 LRS 保管库中保护该 VM。 请参阅将 VM 移到其他资源组的[指南和限制](../azure-resource-manager/management/move-limitations/virtual-machines-move-limitations.md)。

  同一时间只能在一个保管库中保护 VM。 但是，可以在 LRS 保管库中保护新资源组中的 VM，因为它被视为不同的 VM。

  - Azure 备份服务将在 GRS 保管库中保留已备份的恢复点。
  - 你需要付费才能将恢复点保留在 GRS 保管库中（有关详细信息，请参阅 [Azure 备份定价](https://www.azure.cn/pricing/details/backup/index.html)）。
  - 如果需要，你将能够从 GRS 保管库还原 VM。
  - LRS 保管库中对新资源组中 VM 的第一个备份将是初始副本。

<!--Correct in MC: https://www.azure.cn/pricing/details/backup/index.html-->
## <a name="next-steps"></a>后续步骤

[了解](backup-azure-recovery-services-vault-overview.md)恢复服务保管库。
[了解](backup-azure-delete-vault.md)如何删除恢复服务保管库。

