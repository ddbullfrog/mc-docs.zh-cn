---
title: Azure PowerShell - 在 VM 主机上启用端到端加密
description: 如何使用主机加密为 Azure VM 启用端到端加密。
ms.service: virtual-machines
ms.topic: how-to
origin.date: 08/24/2020
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: yes
ms.testdate: 10/19/2020
ms.author: v-yeche
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 43d5f6a7f98f0868c4ad594e91f872885c7b284c
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128353"
---
<!--NOT AVAIABLE ON Mooncake, REQUEST SUBMIT WITHOUT REPLY-->
# <a name="use-the-azure-powershell-module-to-enable-end-to-end-encryption-using-encryption-at-host"></a>使用 Azure PowerShell 模块通过主机加密来启用端到端加密

启用主机加密时，存储在 VM 主机上的数据将静态加密，且已加密的数据将流向存储服务。 有关主机加密以及其他托管磁盘加密类型的概念信息，请参阅[主机加密 - 为 VM 数据启用端到端加密](disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data)。

## <a name="restrictions"></a>限制

[!INCLUDE [virtual-machines-disks-encryption-at-host-restrictions](../../../includes/virtual-machines-disks-encryption-at-host-restrictions.md)]

### <a name="supported-regions"></a>支持的区域

[!INCLUDE [virtual-machines-disks-encryption-at-host-regions](../../../includes/virtual-machines-disks-encryption-at-host-regions.md)]

### <a name="supported-vm-sizes"></a>支持的 VM 大小

[!INCLUDE [virtual-machines-disks-encryption-at-host-suported-sizes](../../../includes/virtual-machines-disks-encryption-at-host-suported-sizes.md)]

你还可以以编程方式找到 VM 大小。 若要了解如何以编程方式检索它们，请参阅[查找支持的 VM 大小](#finding-supported-vm-sizes)部分。

## <a name="prerequisites"></a>先决条件

为了能够为 VM 或虚拟机规模集使用主机加密，必须在订阅上启用该功能。 使用你的订阅 ID 和 Azure 环境名称 (`AzureChinaCloud`) 向 encryptionAtHost@microsoft.com 发送电子邮件，以为你的订阅启用该功能。

<!--CORRECT TO ADD Azure environment name(`AzureChinaCloud`)-->

### <a name="create-an-azure-key-vault-and-diskencryptionset"></a>创建 Azure Key Vault 和 DiskEncryptionSet

启用该功能后，需要设置 Azure Key Vault 和 DiskEncryptionSet（如果尚未设置）。

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-powershell](../../../includes/virtual-machines-disks-encryption-create-key-vault-powershell.md)]

## <a name="enable-encryption-at-host-for-disks-attached-to-vm-and-virtual-machine-scale-sets"></a>为附加到 VM 和虚拟机规模集的磁盘启用主机加密

可以通过在 VM 或使用 API 版本“2020-06-01”和更高版本的虚拟机规模集的 securityProfile 下设置新的属性 EncryptionAtHost，启用主机加密。

`"securityProfile": { "encryptionAtHost": "true" }`

## <a name="example-scripts"></a>示例脚本

### <a name="enable-encryption-at-host-for-disks-attached-to-a-vm-with-customer-managed-keys"></a>使用客户管理的密钥为附加到 VM 的磁盘启用主机加密

使用前面创建的 DiskEncryptionSet 的资源 URI 创建具有托管磁盘的 VM。

替换 `<yourPassword>`、`<yourVMName>`、`<yourVMSize>`、`<yourDESName>`、`<yoursubscriptionID>`、`<yourResourceGroupName>` 和 `<yourRegion>`，然后运行该脚本。

```PowerShell
$password=ConvertTo-SecureString -String "<yourPassword>" -AsPlainText -Force
New-AzResourceGroupDeployment -ResourceGroupName <yourResourceGroupName> `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/EncryptionAtHost/CreateVMWithDisksEncryptedAtHostWithCMK.json" `
-virtualMachineName "<yourVMName>" `
-adminPassword $password `
-vmSize "<yourVMSize>" `
-diskEncryptionSetId "/subscriptions/<yoursubscriptionID>/resourceGroups/<yourResourceGroupName>/providers/Microsoft.Compute/diskEncryptionSets/<yourDESName>" `
-region "<yourRegion>"
```

### <a name="enable-encryption-at-host-for-disks-attached-to-a-vm-with-platform-managed-keys"></a>使用平台管理的密钥为附加 VM 的磁盘启用主机加密

替换 `<yourPassword>``<yourVMName>``<yourVMSize>``<yourResourceGroupName>` 和 `<yourRegion>`，然后运行该脚本。

```PowerShell
$password=ConvertTo-SecureString -String "<yourPassword>" -AsPlainText -Force
New-AzResourceGroupDeployment -ResourceGroupName <yourResourceGroupName> `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/EncryptionAtHost/CreateVMWithDisksEncryptedAtHostWithPMK.json" `
-virtualMachineName "<yourVMName>" `
-adminPassword $password `
-vmSize "<yourVMSize>" `
-region "<yourRegion>"
```

## <a name="finding-supported-vm-sizes"></a>找到支持的 VM 大小

不支持“旧的 VM 大小”。 可以通过以下任一方法查找支持的 VM 大小列表：

调用[资源 SKU API](https://docs.microsoft.com/rest/api/compute/resourceskus/list) 并检查 `EncryptionAtHostSupported` 功能是否设置为 True。

```json
    {
        "resourceType": "virtualMachines",
        "name": "Standard_DS1_v2",
        "tier": "Standard",
        "size": "DS1_v2",
        "family": "standardDSv2Family",
        "locations": [
        "chinanorthEUAP"
        ],
        "capabilities": [
        {
            "name": "EncryptionAtHostSupported",
            "value": "True"
        }
        ]
    }
```

或者，调用 [AzComputeResourceSku](https://docs.microsoft.com/powershell/module/az.compute/get-azcomputeresourcesku?view=azps-3.8.0) PowerShell cmdlet。

<!--CORRECT ON $_.Locations.contains-->

```powershell
$vmSizes=Get-AzComputeResourceSku | where{$_.ResourceType -eq 'virtualMachines' -and $_.Locations.contains('chinaeast')} 

foreach($vmSize in $vmSizes)
{
    foreach($capability in $vmSize.capabilities)
    {
        if($capability.Name -eq 'EncryptionAtHostSupported' -and $capability.Value -eq 'true')
        {
            $vmSize

        }

    }
}
```

## <a name="next-steps"></a>后续步骤

创建并配置这些资源之后，可以使用它们来保护托管磁盘。 以下链接包含示例脚本，每个脚本都有各自的方案，可用于保护托管磁盘。

[Azure 资源管理器模板示例](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/EncryptionAtHost)

<!-- Update_Description: update meta properties, wording update, update link -->