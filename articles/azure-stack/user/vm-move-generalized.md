---
title: 将通用化 VM 从本地移动到 Azure Stack Hub
description: 了解如何将通用化 VM 从本地移动到 Azure Stack Hub。
author: WenJason
ms.topic: how-to
origin.date: 9/8/2020
ms.date: 10/12/2020
ms.author: v-jay
ms.reviewer: kivenkat
ms.lastreviewed: 9/8/2020
ms.openlocfilehash: b9f6a639e93a9fbfedd0e778dc6dba183b4408ec
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91451248"
---
# <a name="move-a-generalized-vm-from-on-premises-to-azure-stack-hub"></a>将通用化 VM 从本地移动到 Azure Stack Hub

可以从本地环境添加虚拟机 (VM) 映像。 可以创建映像作为虚拟硬盘 (VHD)，并将映像上传到 Azure Stack Hub 实例中的存储帐户。 然后可以从 VHD 创建 VM。

通用磁盘映像是使用 Sysprep 准备的用于删除任何唯一信息（如用户帐户）的映像，从而使它能够重复用于创建多个 VM。 在创建 Azure Stack Hub 云操作员规划用作市场项的映像时，通用 VHD 非常合适。

## <a name="how-to-move-an-image"></a>如何移动映像

在准备 VHD 时查找特定于你需求的部分。

#### <a name="windows-vm"></a>[Windows VM](#tab/port-win)

按照[准备好要上传到 Azure 的 Windows VHD 或 VHDX](/virtual-machines/windows/prepare-for-upload-vhd-image)中的步骤，在上传之前正确通用化 VHD。 必须对 Azure Stack Hub 使用 VHD。

#### <a name="linux-vm"></a>[Linux VM](#tab/port-linux)

按照相应的说明，为 Linux OS 使 VHD 通用化：

- [基于 CentOS 的分发版](/virtual-machines/linux/create-upload-centos?toc=%2fvirtual-machines%2flinux%2ftoc.json)
- [Debian Linux](/virtual-machines/linux/debian-create-upload-vhd?toc=%2fvirtual-machines%2flinux%2ftoc.json)
- [Red Hat Enterprise Linux](../operator/azure-stack-redhat-create-upload-vhd.md)
- [SLES 或 openSUSE](/virtual-machines/linux/suse-create-upload-vhd?toc=%2fvirtual-machines%2flinux%2ftoc.json)
- [Ubuntu Server](/virtual-machines/linux/create-upload-ubuntu?toc=%2fvirtual-machines%2flinux%2ftoc.json)

---

## <a name="verify-your-vhd"></a>验证 VHD

[!INCLUDE [Verify VHD](../includes/user-compute-verify-vhd.md)]
## <a name="upload-to-a-storage-account"></a>上传到存储帐户

[!INCLUDE [Upload to a storage account](../includes/user-compute-upload-vhd.md)]

## <a name="create-the-image-in-azure-stack-hub"></a>在 Azure Stack Hub 中创建映像

[!INCLUDE [Create the image in Azure Stack Hub](../includes/user-compute-create-image.md)]

## <a name="next-steps"></a>后续步骤

[将 VM 移动到 Azure Stack Hub 概述](vm-move-overview.md)
