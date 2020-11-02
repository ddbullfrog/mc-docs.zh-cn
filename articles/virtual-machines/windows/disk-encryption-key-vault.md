---
title: 创建和配置用于 Azure 磁盘加密的密钥保管库
description: 本文介绍创建和配置用于 Windows VM 上的 Azure 磁盘加密的密钥保管库的步骤。
ms.service: virtual-machines
ms.subservice: security
ms.topic: how-to
origin.date: 08/06/2019
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 10/19/2020
ms.author: v-yeche
ms.custom: seodec18
ms.openlocfilehash: db59dcdb87adaf38ba3da5ee99d31fb4646ed518
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104339"
---
# <a name="creating-and-configuring-a-key-vault-for-azure-disk-encryption"></a>创建和配置用于 Azure 磁盘加密的密钥保管库

Azure Disk Encryption 使用 Azure Key Vault 来控制和管理磁盘加密密钥和机密。  有关 Key Vault 的详细信息，请参阅 [Azure Key Vault 入门](../../key-vault/general/overview.md)和[保护 Key Vault](../../key-vault/general/secure-your-key-vault.md)。 

> [!WARNING]
> - 如果之前是使用 Azure 磁盘加密与 Azure AD 来加密 VM，则必须继续使用此选项来加密 VM。 有关详细信息，请参阅[使用 Azure AD 创建和配置用于 Azure 磁盘加密的 Key Vault（以前版本）](disk-encryption-key-vault-aad.md)。

创建和配置用于 Azure 磁盘加密的 Key Vault 需要三个步骤：

> [!Note]
> 必须在 Azure Key Vault 访问策略设置中选择该选项，才能为卷加密启用对 Azure 磁盘加密的访问。 如果已在密钥保管库上启用防火墙，则必须转到密钥保管库上的“网络”选项卡并启用对 Azure 受信任的服务的访问。 

1. 创建资源组（如果需要）。
2. 创建密钥保管库。 
3. 设置密钥保管库高级访问策略。

以下快速入门说明了这些步骤：

- [使用 Azure CLI 创建和加密 Windows VM](disk-encryption-cli-quickstart.md)
- [使用 Azure PowerShell 创建和加密 Windows VM](disk-encryption-powershell-quickstart.md)

还可以根据需要生成或导入密钥加密密钥 (KEK)。

> [!Note]
> 本文中的步骤在 [Azure 磁盘加密先决条件 CLI 脚本](https://github.com/ejarvi/ade-cli-getting-started)和 [Azure 磁盘加密先决条件 PowerShell 脚本](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts)中自动执行。

## <a name="install-tools-and-connect-to-azure"></a>安装工具并连接到 Azure

可以使用 [Azure CLI](https://docs.azure.cn/cli/)、[Azure PowerShell Az 模块](https://docs.microsoft.com/powershell/azure/)或 [Azure 门户](https://portal.azure.cn)来完成本文中的步骤。

虽然可以通过浏览器访问门户，但 Azure CLI 和 Azure PowerShell 需要本地安装；有关详细信息，请参阅[适用于 Windows 的 Azure 磁盘加密：安装工具](disk-encryption-windows.md#install-tools-and-connect-to-azure)。

### <a name="connect-to-your-azure-account"></a>连接到 Azure 帐户

使用 Azure CLI 或 Azure PowerShell 之前，必须先连接到 Azure 订阅。 为此，可以[使用 Azure CLI 登录](https://docs.azure.cn/cli/authenticate-azure-cli)、[使用 Azure Powershell 登录](https://docs.microsoft.com/powershell/azure/authenticate-azureps?view=azps-2.5.0)，或在出现提示时向 Azure 门户提供凭据。

```azurecli
az cloud set -n AzureChinaCloud
az login
```

```powershell
Connect-AzAccount -Environment AzureChinaCloud
```

[!INCLUDE [disk-encryption-key-vault](../../../includes/disk-encryption-key-vault.md)]

## <a name="next-steps"></a>后续步骤

- [Azure 磁盘加密先决条件 CLI 脚本](https://github.com/ejarvi/ade-cli-getting-started)
- [Azure 磁盘加密先决条件 PowerShell 脚本](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts)
- 了解 [Windows VM 上的 Azure 磁盘加密方案](disk-encryption-windows.md)
- 了解如何[对 Azure 磁盘加密进行故障排除](disk-encryption-troubleshooting.md)
- 阅读 [Azure 磁盘加密示例脚本](disk-encryption-sample-scripts.md)

<!-- Update_Description: update meta properties, wording update, update link -->