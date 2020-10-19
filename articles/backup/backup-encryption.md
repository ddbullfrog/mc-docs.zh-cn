---
title: Azure 备份中的加密
description: 了解 Azure 备份中的加密功能如何帮助你保护备份数据并满足企业的安全需求。
ms.topic: conceptual
author: Johnnytechn
ms.author: v-johya
ms.date: 09/28/2020
ms.custom: references_regions
ms.openlocfilehash: 1d3cf9956fce1934379b470c8f275714c4b73563
ms.sourcegitcommit: 80567f1c67f6bdbd8a20adeebf6e2569d7741923
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91871322"
---
# <a name="encryption-in-azure-backup"></a>Azure 备份中的加密

将所有备份数据存储在云中时，会使用 Azure 存储加密自动对其进行加密，这有助于履行安全性与合规性承诺。 此静态数据将使用 256 位 AES 加密法（可用的最强大块加密法之一）以透明方式进行加密，并符合 FIPS 140-2 规范。 除了静态加密以外，所有传输中的备份数据将通过 HTTPS 传输。 这些数据始终保留在 Azure 主干网络上。

## <a name="levels-of-encryption-in-azure-backup"></a>Azure 备份中的加密级别

Azure 备份提供两个级别的加密：

- **恢复服务保管库中的数据加密**
  - **使用平台管理的密钥**：默认情况下，所有数据将使用平台托管的密钥进行加密。 无需从你的终端执行任何明确操作即可实现此加密。 这种加密适用于要备份到恢复服务保管库的所有工作负荷。
- **特定于要备份的工作负荷的加密**  
  - **Azure 虚拟机备份**：Azure 备份支持对特定 VM 进行备份，这些 VM 包含的磁盘使用[平台管理的密钥](/virtual-machines/windows/disk-encryption#platform-managed-keys)以及你拥有和管理的[客户管理的密钥](/virtual-machines/windows/disk-encryption#customer-managed-keys)进行加密。 此外，还可以备份已使用 [Azure 磁盘加密](backup-azure-vms-encryption.md#encryption-support-using-ade)将其 OS 磁盘或数据磁盘加密的 Azure 虚拟机。 ADE 使用适用于 Windows VM 的 BitLocker 以及适用于 Linux VM 的 DM-Crypt 来执行来宾内部加密。

## <a name="next-steps"></a>后续步骤

- [静态数据的 Azure 存储加密](/storage/common/storage-service-encryption)
- [Azure 备份常见问题解答](backup-azure-backup-faq.md#encryption)，解答有关加密的任何问题

