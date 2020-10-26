---
title: 适用于 Linux 的 Azure 磁盘加密 (Microsoft.Azure.Security.AzureDiskEncryptionForLinux) | Azure
description: 使用虚拟机扩展将适用于 Linux 的 Azure 磁盘加密部署到虚拟机。
services: virtual-machines-linux
documentationcenter: ''
author: rockboyfor
manager: digimobile
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
origin.date: 06/10/2019
ms.date: 11/11/2019
ms.author: v-yeche
ms.openlocfilehash: c1b7505bae2d4dee1c0e3b19a5e6f883247fcf7b
ms.sourcegitcommit: 753c74533aca0310dc7acb621cfff5b8993c1d20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92211613"
---
# <a name="azure-disk-encryption-for-linux-microsoftazuresecurityazurediskencryptionforlinux"></a>适用于 Linux 的 Azure 磁盘加密 (Microsoft.Azure.Security.AzureDiskEncryptionForLinux)

## <a name="overview"></a>概述

Azure 磁盘加密利用 Linux 中的 dm-crypt 子系统在选定的 Azure Linux 发行版上提供完整磁盘加密。 此解决方案与 Azure Key Vault 集成，用于管理磁盘加密密钥和机密。

<!--Not Available on [select Azure Linux distributions](https://docs.azure.cn/security/azure-security-disk-encryption-faq)-->

## <a name="prerequisites"></a>先决条件

有关先决条件的完整列表，请参阅[适用于 Linux VM 的 Azure 磁盘加密](../linux/disk-encryption-overview.md)，特别是以下部分：

- [适用于 Linux VM 的 Azure 磁盘加密](../linux/disk-encryption-overview.md#supported-vms-and-operating-systems)
- [其他 VM 要求](../linux/disk-encryption-overview.md#additional-vm-requirements)
- [网络要求](../linux/disk-encryption-overview.md#networking-requirements)

## <a name="extension-schemata"></a>扩展架构

Azure 磁盘加密有两种架构：v1.1，一种不使用 Azure Active Directory (AAD) 属性的较新推荐架构；v0.1，一种需要 AAD 属性的较旧架构。 你必须使用与所使用的扩展对应的架构版本：架构 v1.1 用于 AzureDiskEncryptionForLinux 扩展版本 1.1，架构 v0.1 用于 AzureDiskEncryptionForLinux 扩展版本 0.1。
### <a name="schema-v11-no-aad-recommended"></a>架构 v1.1：无 AAD（推荐）

建议使用 v1.1 架构，它不需要 Azure Active Directory 属性。

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2015-06-15",
  "location": "[location]",
  "properties": {
        "publisher": "Microsoft.Azure.Security",
        "settings": {
          "DiskFormatQuery": "[diskFormatQuery]",
          "EncryptionOperation": "[encryptionOperation]",
          "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
          "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
          "KeyVaultURL": "[keyVaultURL]",
          "SequenceVersion": "sequenceVersion]",
          "VolumeType": "[volumeType]"
        },
        "type": "AzureDiskEncryptionForLinux",
        "typeHandlerVersion": "[extensionVersion]"
  }
}
```

### <a name="schema-v01-with-aad"></a>架构 v0.1：使用 AAD 

0\.1 版架构需要 `aadClientID` 和 `aadClientSecret` 或 `AADClientCertificate`。

使用 `aadClientSecret`：

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2015-06-15",
  "location": "[location]",
  "properties": {
    "protectedSettings": {
      "AADClientSecret": "[aadClientSecret]",
      "Passphrase": "[passphrase]"
    },
    "publisher": "Microsoft.Azure.Security",
    "settings": {
      "AADClientID": "[aadClientID]",
      "DiskFormatQuery": "[diskFormatQuery]",
      "EncryptionOperation": "[encryptionOperation]",
      "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
      "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
      "KeyVaultURL": "[keyVaultURL]",
      "SequenceVersion": "sequenceVersion]",
      "VolumeType": "[volumeType]"
    },
    "type": "AzureDiskEncryptionForLinux",
    "typeHandlerVersion": "[extensionVersion]"
  }
}
```

使用 `AADClientCertificate`：

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2015-06-15",
  "location": "[location]",
  "properties": {
    "protectedSettings": {
      "AADClientCertificate": "[aadClientCertificate]",
      "Passphrase": "[passphrase]"
    },
    "publisher": "Microsoft.Azure.Security",
    "settings": {
      "AADClientID": "[aadClientID]",
      "DiskFormatQuery": "[diskFormatQuery]",
      "EncryptionOperation": "[encryptionOperation]",
      "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
      "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
      "KeyVaultURL": "[keyVaultURL]",
      "SequenceVersion": "sequenceVersion]",
      "VolumeType": "[volumeType]"
    },
    "type": "AzureDiskEncryptionForLinux",
    "typeHandlerVersion": "[extensionVersion]"
  }
}
```

### <a name="property-values"></a>属性值

| 名称 | 值/示例 | 数据类型 |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | date |
| publisher | Microsoft.Azure.Security | string |
| type | AzureDiskEncryptionForLinux | string |
| typeHandlerVersion | 0.1、1.1 | int |
| （0.1 版架构）AADClientID | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | GUID | 
| （0.1 版架构）AADClientSecret | password | string |
| （0.1 版架构）AADClientCertificate | thumbprint | string |
| DiskFormatQuery | {"dev_path":"","name":"","file_system":""} | JSON 字典 |
| EncryptionOperation | EnableEncryption, EnableEncryptionFormatAll | string | 
| KeyEncryptionAlgorithm | 'RSA-OAEP', 'RSA-OAEP-256', 'RSA1_5' | string |
| KeyEncryptionKeyURL | url | string |
| （可选）KeyVaultURL | url | string |
| 通行短语 | password | string | 
| SequenceVersion | uniqueidentifier | string |
| VolumeType | OS, Data, All | string |

## <a name="template-deployment"></a>模板部署

有关模板部署的示例，请参阅[在正在运行的 Linux VM 上启用加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm)。

>[!NOTE]
> 必须修改从 GitHub 存储库“azure-quickstart-templates”下载或参考的模板，以适应 Azure 中国云环境。 例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；必要时更改某些不受支持的 VM 映像、VM 大小、SKU 以及资源提供程序的 API 版本。

## <a name="azure-cli-deployment"></a>Azure CLI 部署

可以在最新 [Azure CLI 文档](https://docs.azure.cn/cli/vm/encryption?view=azure-cli-latest)中找到相关说明。 

## <a name="troubleshoot-and-support"></a>故障排除和支持

<!-- Not Available on ### Troubleshoot-->
<!-- Not Available on [Azure Disk Encryption troubleshooting guide](../../security/azure-security-disk-encryption-tsg.md)-->

### <a name="support"></a>支持

如果对本文中的任何观点存在疑问，可以联系 [Azure 支持](https://support.azure.cn/support/contact/)上的 Azure 专家。 或者，也可以提出 Azure 支持事件。 请转到 [Azure 支持站点](https://support.azure.cn/support/support-azure/)提交请求。 有关使用 Azure 支持的信息，请阅读 [Azure 支持常见问题](https://www.azure.cn/support/faq/)。

## <a name="next-steps"></a>后续步骤

有关 VM 扩展的详细信息，请参阅[适用于 Linux 的虚拟机扩展和功能](features-linux.md)。

<!-- Update_Description: wording update, update meta properties -->
