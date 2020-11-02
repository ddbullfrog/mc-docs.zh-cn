---
title: 在 Azure 中创建虚拟机时不支持 VHD | Azure
description: 本文有助于在 Azure 中运行虚拟机时更正 VHD 错误。
services: virtual-machines
manager: dcscontentpm
ms.service: virtual-machines
ms.assetid: 5488aba9-c3da-435d-b4a5-63470f455b07
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure
origin.date: 06/29/2020
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: fc03726301f0436dae36b360f56b1d2ec9546979
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104791"
---
# <a name="vhd-is-not-supported-when-you-create-a-virtual-machine-in-azure"></a>在 Azure 中创建虚拟机时不支持 VHD

本文有助于在 Windows 或 Linux 中运行虚拟机时更正 VHD 错误。

## <a name="symptoms"></a>症状

使用上传的 VHD 在 Azure 中创建虚拟机时，部署失败并返回以下错误消息： 

```
New-AzureRmVM : Long running operation failed with status 'Failed'.
ErrorCode: InvalidVhd
ErrorMessage: The specified cookie value in VHD footer indicates that disk 'diskname' with blob https://xxxxxx.blob.core.chinacloudapi.cn/vhds/samplename.vhd is not a supported VHD. Disk is expected to have cookie value 'conectix'.
```

## <a name="cause"></a>原因

此问题是由以下原因之一导致的：

- VHD 不符合 1 MB 对齐（偏移）规范。 支持的磁盘大小应为 1 MB * N。例如，磁盘大小应为 102,401 MB。
- VHD 已损坏或不受支持。 

## <a name="resolution"></a>解决方法

> [!NOTE]
> 若要执行以下修复，客户需要在将 VHD 上传到 Azure 之前执行这些步骤。

若要解决此问题，请调整磁盘大小以符合 1 MB 一致性：

- 若要解决 Windows 中的问题，请使用 [Resize-VHD PowerShell cmdlet](https://docs.microsoft.com/powershell/module/hyper-v/resize-vhd)。 请注意，Resize VHD 不是 Azure PowerShell cmdlet。

    1. [在 Windows Server 上安装 Hyper-V 角色](https://docs.microsoft.com/windows-server/virtualization/hyper-v/get-started/install-the-hyper-v-role-on-windows-server)
    1. [将虚拟磁盘转换为固定大小的 VHD](../windows/prepare-for-upload-vhd-image.md#convert-the-virtual-disk-to-a-fixed-size-vhd)

- 若要解决 Linux 中的问题，请使用 [qemu img 命令](../linux/create-upload-generic.md)。

有关如何创建和上传 VHD 以创建 Azure VM 的更多信息，请参阅以下文章：

- [使用 Azure CLI 1.0 上传自定义磁盘映像并从其创建 Linux VM](../linux/upload-vhd.md)
- [创建 Windows Server VHD 并将其上传到 Azure](../windows/upload-generalized-managed.md)

持续出现的问题可能表明 VHD 已损坏。 在这种情况下，我们建议你从头开始重新生成 VHD。

有关详细信息，请参阅以下文章：

- [关于 VHD](../managed-disks-overview.md)

<!-- Update_Description: update meta properties, wording update, update link -->