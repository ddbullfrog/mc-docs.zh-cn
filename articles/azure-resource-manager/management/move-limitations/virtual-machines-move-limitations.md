---
title: 将 Azure VM 移到新的订阅或资源组
description: 使用 Azure 资源管理器将虚拟机移到新的资源组或订阅。
ms.topic: conceptual
origin.date: 08/31/2020
author: rockboyfor
ms.date: 09/21/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 75bb3a0db96199d17ba64149f6fd22751f2add82
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146771"
---
# <a name="move-guidance-for-virtual-machines"></a>针对虚拟机的移动指南

本文介绍当前不支持的方案以及移动使用备份的虚拟机的步骤。

## <a name="scenarios-not-supported"></a>不支持的方案

以下方案尚不受支持：

<!--Not Available on Availability Zones-->
* 无法移动具有标准 SKU 负载均衡器或标准 SKU 公共 IP 的虚拟机规模集。
* 无法跨订阅移动基于附加了计划的市场资源创建的虚拟机。 在当前订阅中取消预配虚拟机，并在新的订阅中重新部署虚拟机。
* 如果没有移动虚拟网络中的所有资源，则无法将现有虚拟网络中的虚拟机移到新订阅。
<!--MOONCAKE: Not Available on Low priority-->
    
* 可用性集中的虚拟机不能单独移动。

## <a name="azure-disk-encryption"></a>Azure 磁盘加密

无法移动与密钥保管库集成的虚拟机以实现[适用于 Linux VM 的 Azure 磁盘加密](../../../virtual-machines/linux/disk-encryption-overview.md)或[适用于 Windows VM 的 Azure 磁盘加密](../../../virtual-machines/windows/disk-encryption-overview.md)。 若要移动 VM，必须禁用加密。

```azurecli
az vm encryption disable --resource-group demoRG --name myVm1
```

```powershell
Disable-AzVMDiskEncryption -ResourceGroupName demoRG -VMName myVm1
```

## <a name="virtual-machines-with-azure-backup"></a>使用 Azure 备份的虚拟机

若要移动配置了 Azure 备份的虚拟机，必须从保管库中删除还原点。

如果为虚拟机启用了[软删除](../../../backup/backup-azure-security-feature-cloud.md)，则在保留这些还原点的情况下，你将无法移动虚拟机。 请[禁用软删除](../../../backup/backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete)，或在删除还原点后等待 14 天。

### <a name="portal"></a>门户

1. 暂时停止备份并保留备份数据。
2. 若要移动配置了 Azure 备份的虚拟机，请执行以下步骤：

    1. 找到虚拟机的位置。
    2. 找到包含以下命名模式的资源组：`AzureBackupRG_<VM location>_1`。 例如，名称的格式为 AzureBackupRG_chinanorth2_1。
    3. 在 Azure 门户中，查看“显示隐藏的类型”。
    4. 查找类型为 Microsoft.Compute/restorePointCollections 的资源，其命名模式为 `AzureBackup_<name of your VM that you're trying to move>_###########`。
    5. 删除此资源。 此操作仅删除即时恢复点，不删除保管库中的备份数据。
    6. 删除操作完成后，可以移动虚拟机。

3. 将 VM 移到目标资源组。
4. 恢复备份。

### <a name="powershell"></a>PowerShell

1. 找到虚拟机的位置。

1. 找到采用以下命名模式的资源组：`AzureBackupRG_<VM location>_1`。 例如，名称可以为 `AzureBackupRG_chinanorth2_1`。

1. 使用以下命令获取还原点集合。

    ```azurepowershell
    $RestorePointCollection = Get-AzResource -ResourceGroupName AzureBackupRG_<VM location>_1 -ResourceType Microsoft.Compute/restorePointCollections
    ```

1. 删除此资源。 此操作仅删除即时恢复点，不删除保管库中的备份数据。

    ```azurepowershell
    Remove-AzResource -ResourceId $RestorePointCollection.ResourceId -Force
    ```

### <a name="azure-cli"></a>Azure CLI

1. 找到虚拟机的位置。

1. 找到采用以下命名模式的资源组：`AzureBackupRG_<VM location>_1`。 例如，名称可以为 `AzureBackupRG_chinanorth2_1`。

1. 使用以下命令获取还原点集合。

    ```azurecli
    az resource list -g AzureBackupRG_<VM location>_1 --resource-type Microsoft.Compute/restorePointCollections
    ```

1. 查找命名模式为 `AzureBackup_<VM name>_###########` 的资源的资源 ID

1. 删除此资源。 此操作仅删除即时恢复点，不删除保管库中的备份数据。

    ```azurecli
    az resource delete --ids /subscriptions/<sub-id>/resourceGroups/<resource-group>/providers/Microsoft.Compute/restorePointCollections/<name>
    ```

## <a name="next-steps"></a>后续步骤

* 有关用于移动资源的命令，请参阅[将资源移到新资源组或订阅](../move-resource-group-and-subscription.md)。

<!--Not Available on  [Recovery Services limitations](../../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure-resource-manager/toc.json)-->

<!-- Update_Description: update meta properties, wording update, update link -->
