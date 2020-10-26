---
title: 通过使用 LVM（逻辑卷管理器）的 chroot 恢复 Linux VM - Azure VM
description: 使用 LVM 恢复 Linux VM。
services: virtual-machines-linux
documentationcenter: ''
manager: spogge
editor: ''
tags: Linux chroot LVM
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
origin.date: 11/24/2019
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: yes
ms.testdate: 10/19/2020
ms.author: v-yeche
ms.openlocfilehash: fa51719e0f2047b069f08bbf66479fdb7efc9fe4
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92127898"
---
# <a name="troubleshooting-a-linux-vm-when-there-is-no-access-to-the-azure-serial-console-and-the-disk-layout-is-using-lvm-logical-volume-manager"></a>当无权访问 Azure 串行控制台且磁盘布局使用 LVM（逻辑卷管理器）时，对 Linux VM 进行故障排除

<!--Not Avaialble on no access to the Azure serial console and-->

本故障排除指南可帮助解决以下场景中出现的问题：Linux VM 不启动、无法建立 SSH 连接，并且在底层文件系统布局中配置了 LVM（逻辑卷管理器）。

## <a name="take-snapshot-of-the-failing-vm"></a>创建有故障 VM 的快照

创建受影响 VM 的快照。 

然后，该快照将附加到 **救援** VM。 根据 [此处](../linux/snapshot-copy-managed-disk.md#use-azure-portal)的说明了解如何创建 **快照** 。

## <a name="create-a-rescue-vm"></a>创建救援 VM
一般情况下，我们建议使用相同或类似的操作系统版本创建救援 VM。 使用与受影响 VM 相同的 **区域** 和 **资源组**

## <a name="connect-to-the-rescue-vm"></a>连接到救援 VM
使用 SSH 连接到 **救援** VM。 使用以下命令提升权限并成为超级用户

`sudo su -`

## <a name="attach-the-disk"></a>附加磁盘
将磁盘附加到基于前面创建的快照构建的 **救援** VM。

在 Azure 门户 -> 选择 **救援** VM ->“磁盘”  

:::image type="content" source="./media/chroot-logical-volume-manager/create-disk-from-snap.png" alt-text="创建磁盘":::

填写字段。 为新磁盘命名，并选择与快照、受影响 VM 和救援 VM 相同的资源组。

“源类型”为“快照”。  
“源快照”是前面创建的 **快照** 的名称。 

:::image type="content" source="./media/chroot-logical-volume-manager/create-disk-from-snap-2.png" alt-text="创建磁盘":::

为附加的磁盘创建装入点。

`mkdir /rescue`

运行 **fdisk -l** 命令来验证是否已附加快照磁盘，并列出所有可用的设备和分区

`fdisk -l`

在大多数情况下，附加的快照磁盘将显示为 **/dev/sdc** ，其中显示了 **/dev/sdc1** 和 **/dev/sdc2** 这两个分区

:::image type="content" source="./media/chroot-logical-volume-manager/fdisk-output-sdc.png" alt-text="创建磁盘":::

**\*** 表示启动分区；将会装载这两个分区。

运行 **lsblk** 命令查看受影响 VM 的 LVM

`lsblk`

:::image type="content" source="./media/chroot-logical-volume-manager/lsblk-output-mounted.png" alt-text="创建磁盘":::

验证是否显示受影响 VM 中的 LVM。
如果未显示，请使用以下命令启用 LVM，然后重新运行 **lsblk** 。
在继续操作之前，请确保附加磁盘中的 LVM 已显示。

```
vgscan --mknodes
vgchange -ay
lvscan
mount -a
lsblk
```

找到包含 /（根）分区的逻辑卷的装载路径。 其中包含类似于 /etc/default/grub 的配置文件

本示例采用上述 **lsblk** 命令的输出。 **rootvg-rootlv** 是要装载的正确 **根** LV，可在下一条命令中使用它。

下一条命令的输出将显示 **根** LV 的装载路径

`pvdisplay -m | grep -i rootlv`

:::image type="content" source="./media/chroot-logical-volume-manager/locate-rootlv.png" alt-text="创建磁盘":::

继续在 /rescue 目录中装载此设备

`mount /dev/rootvg/rootlv /rescue`

在 /rescue/boot 中装载设置了 **启动标志** 的分区

`
mount /dev/sdc1 /rescue/boot
`

使用 **lsblk** 命令验证现在是否已正确装载附加磁盘的文件系统

:::image type="content" source="./media/chroot-logical-volume-manager/lsblk-output-1.png" alt-text="创建磁盘":::

或 **df -Th** 命令

:::image type="content" source="./media/chroot-logical-volume-manager/df-output.png" alt-text="创建磁盘":::

## <a name="gaining-chroot-access"></a>获取 chroot 访问权限

获取 **chroot** 访问权限，以便能够执行各种修复。此操作根据每个 Linux 分发版而略有差异。

```
 cd /rescue
 mount -t proc proc proc
 mount -t sysfs sys sys/
 mount -o bind /dev dev/
 mount -o bind /dev/pts dev/pts/
 chroot /rescue
```

如果出现类似于下面的错误：

“chroot: 无法运行命令 '/bin/bash':  无此类文件或目录”

请尝试装载 **usr** 逻辑卷

`
mount  /dev/mapper/rootvg-usrlv /rescue/usr
`

> [!TIP]
> 在 **chroot** 环境中执行命令时请注意，这些命令是针对附加的 OS 磁盘运行的，而不是针对本地 **救援** VM 运行的。 

命令可用于安装、删除和更新软件。 对 VM 进行故障排除以修复错误。

执行 lsblk 命令/立即修复/修复或启动已启动:::image type="content" source="./media/chroot-logical-volume-manager/chrooted.png" alt-text="创建磁盘":::

## <a name="perform-fixes"></a>执行修复

### <a name="example-1---configure-the-vm-to-boot-from-a-different-kernel"></a>示例 1 - 将 VM 配置为从其他内核启动

一个常见的场景是强制 VM 从以前的内核启动，因为当前安装的内核可能已损坏，或者升级未正确完成。

```
cd /boot/grub2

grep -i linux grub.cfg

grub2-editenv list

grub2-set-default "CentOS Linux (3.10.0-1062.1.1.el7.x86_64) 7 (Core)"

grub2-editenv list

grub2-mkconfig -o /boot/grub2/grub.cfg
```

*演练*

**grep** 命令列出 **grub.cfg** 能够识别的内核。
:::image type="content" source="./media/chroot-logical-volume-manager/kernels.png" alt-text="创建磁盘":::

**grub2-editenv list** 显示下一次启动时要加载的内核 :::image type="content" source="./media/chroot-logical-volume-manager/kernel-default.png" alt-text="创建磁盘":::

**grub2-set-default** 用于切换到另一内核 :::image type="content" source="./media/chroot-logical-volume-manager/grub2-set-default.png" alt-text="创建磁盘":::

**grub2-editenv list** 显示下一次启动时要加载的内核 :::image type="content" source="./media/chroot-logical-volume-manager/kernel-new.png" alt-text="创建磁盘":::

**grub2-mkconfig** 使用所需的版本重建 grub.cfg :::image type="content" source="./media/chroot-logical-volume-manager/grub2-mkconfig.png" alt-text="创建磁盘":::

### <a name="example-2---upgrade-packages"></a>示例 2 - 升级包

失败的内核升级可能会导致 VM 无法启动。
装载所有逻辑卷，以便能够删除或重新安装包

运行 **lvs** 命令来验证哪些 **LV** 可供装载；已迁移的或来自另一个云提供商的每个 VM 因配置而异。

退出 **chroot** 环境并装载所需的 **LV**

:::image type="content" source="./media/chroot-logical-volume-manager/advanced.png" alt-text="创建磁盘":::

现在，运行以下命令再次访问 **chroot** 环境

`chroot /rescue`

所有 LV 应显示为已装载的分区

:::image type="content" source="./media/chroot-logical-volume-manager/chroot-all-mounts.png" alt-text="创建磁盘":::

查询安装的 **内核**

:::image type="content" source="./media/chroot-logical-volume-manager/rpm-kernel.png" alt-text="创建磁盘":::

可根据需要删除或升级 **内核**
:::image type="content" source="./media/chroot-logical-volume-manager/rpm-remove-kernel.png" alt-text="创建磁盘":::

<!--Not Available on ### Example 3 - enable Serial Console-->
<!--Not Available on Serial Console-->

### <a name="example-3---kernel-loading-with-problematic-lvm-swap-volume"></a>示例 3 - LVM 交换卷出问题的内核加载

VM 可能无法完全启动，因此会进入 **dracut** 提示。
有关故障的更多详细信息，可从 Azure 串行控制台查找，也可导航到 Azure 门户 -> 启动诊断 -> 串行日志进行查找

可能出现类似于下面的错误：

```
[  188.000765] dracut-initqueue[324]: Warning: /dev/VG/SwapVol does not exist
         Starting Dracut Emergency Shell...
Warning: /dev/VG/SwapVol does not exist
```

在此示例中配置了 grub.cfg，以加载名为 **rd.lvm.lv=VG/SwapVol** 的 LV，VM 无法找到此项。 此行显示了如何加载引用 LV SwapVol 的内核

```
[    0.000000] Command line: BOOT_IMAGE=/vmlinuz-3.10.0-1062.4.1.el7.x86_64 root=/dev/mapper/VG-OSVol ro console=tty0 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0 biosdevname=0 crashkernel=256M rd.lvm.lv=VG/OSVol rd.lvm.lv=VG/SwapVol nodmraid rhgb quiet
[    0.000000] e820: BIOS-provided physical RAM map:
```

 从 /etc/default/grub 配置中删除有问题的 LV 并重新生成 grub2.cfg

## <a name="exit-chroot-and-swap-the-os-disk"></a>退出 chroot 并交换 OS 磁盘

修复问题后，请继续从救援 VM 中卸载并分离磁盘，使其能够与受影响的 VM OS 磁盘交换。

```
exit
cd /
umount /rescue/proc/
umount /rescue/sys/
umount /rescue/dev/pts
umount /rescue/dev/
umount /rescue/boot
umount /rescue
```

从救援 VM 中分离磁盘，并执行磁盘交换。

在门户的“磁盘”中选择 VM，然后选择“分离”
 

保存更改:::image type="content" source="./media/chroot-logical-volume-manager/save-detach.png" alt-text="创建磁盘"::: 

磁盘现在可用，并可与受影响 VM 的原始 OS 磁盘交换。

在 Azure 门户中导航到有故障的 VM，然后选择“磁盘” -> “交换 OS 磁盘”
:::image type="content" source="./media/chroot-logical-volume-manager/swap-disk.png" alt-text="创建磁盘":::  

填写字段。在“选择磁盘”中选择刚刚在上一步骤中分离的快照磁盘。  受影响 VM 的名称也是必填的。然后选择“确定” 

:::image type="content" source="./media/chroot-logical-volume-manager/new-osdisk.png" alt-text="创建磁盘"::: 

如果 VM 正在运行，则“磁盘交换”操作会将其关闭，并在完成后重新启动 VM。

<!--Not Available on [Azure Serial Console]( https://docs.azure.cn/virtual-machines/troubleshooting/serial-console-linux)-->

<!-- Update_Description: update meta properties, wording update, update link -->