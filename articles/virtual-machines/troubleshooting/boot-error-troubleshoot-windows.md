---
title: Azure 虚拟机关闭时停滞在“正在重启”、“正在关闭”或“正在停止服务”状态 | Azure
description: 本文帮助你排查 Azure Windows 虚拟机中的服务错误。
services: virtual-machines-windows
manager: dcscontentpm
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
origin.date: 12/19/2019
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: bb7aae1e712b6938115bea8973cbc8b24b42777f
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104646"
---
# <a name="azure-windows-vm-shutdown-is-stuck-on-restarting-shutting-down-or-stopping-services"></a>Azure Windows VM 关闭时停滞在“正在重启”、“正在关闭”或“正在停止服务”状态

本文提供的步骤可解决在重新启动 Azure 中的 Windows 虚拟机 (VM) 时，可能遇到的“正在重启”、“正在关闭”或“正在停止服务”消息的问题。

## <a name="symptoms"></a>症状

使用[启动诊断](./boot-diagnostics.md)查看 VM 的屏幕截图时，可能会看到屏幕截图显示消息“正在重启”、“正在关闭”或“正在停止服务”。

:::image type="content" source="./media/boot-error-troubleshooting-windows/restart-shut-down-stop-service.png" alt-text="“正在重启”、“正在关闭”或“正在停止服务”屏幕":::

## <a name="cause"></a>原因

Windows 使用关闭进程来执行系统维护操作，并处理更新、角色和功能等方面的更改。 建议不要中断此关键进程，而是让它完成。 根据更新/更改数量和 VM 大小，此进程可能会花费较长的时间。 如果停止该进程，OS 可能会损坏。 仅当该进程耗时过长时才将其中断。

## <a name="solution"></a>解决方案

<!--Not Avaialable on ### Collect a Process memory dump-->
<!--We can not access the VM and run the following command to check the process-->
<!--While the Serial Console can be access on Azure global site-->
### <a name="collect-an-os-memory-dump"></a>收集 OS 内存转储

<!--Not Available on If the issue does not resolve after waiting for the changes to process-->

需要收集内存转储文件并与支持人员联系。 若要收集转储文件，请执行以下步骤：

**将 OS 磁盘附加到恢复 VM**

1. 拍摄受影响的 VM 的 OS 磁盘的快照作为备份。 有关详细信息，请参阅[拍摄磁盘快照](../windows/snapshot-copy-managed-disk.md)。

2. [将 OS 磁盘附加到恢复 VM](./troubleshoot-recovery-disks-portal-windows.md)。

3. 通过远程桌面连接到恢复 VM。

4. 如果 OS 磁盘已加密，则必须先关闭加密，然后才能进入下一步。 有关详细信息，请参阅[在无法启动的 VM 中解密加密的 OS 磁盘](./troubleshoot-bitlocker-boot-error.md#solution)。

**找到转储文件并提交支持票证**

1. 在恢复 VM 上，转到附加的 OS 磁盘中的 Windows 文件夹。 如果分配给附加 OS 磁盘的驱动器号为 F，则需转到 F:\Windows。

2. 找到 memory.dmp 文件，然后使用该转储文件[提交支持票证](https://support.azure.cn/support/support-azure/)。

如果找不到转储文件，请转到下一步以启用转储日志和串行控制台。

**启用转储日志和串行控制台**

若要启用转储日志和串行控制台，请运行以下脚本。

1. 打开提升的命令提示符会话（以管理员身份运行）。

2. 运行以下脚本：

    在此脚本中，假定分配给附加 OS 磁盘的驱动器号为 F。将其替换为 VM 中的相应值。

    ```
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REM Enable Serial Console
    bcdedit /store F:\boot\bcd /set {bootmgr} displaybootmenu yes
    bcdedit /store F:\boot\bcd /set {bootmgr} timeout 5
    bcdedit /store F:\boot\bcd /set {bootmgr} bootems yes
    bcdedit /store F:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
    bcdedit /store F:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200

    REM Suggested configuration to enable OS Dump
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump / t REG_DWORD /d 1 /f

    reg unload HKLM\BROKENSYSTEM
    ```

3. 确认磁盘上有足够的空间来分配与 RAM 一样多的内存，具体取决于为此 VM 选择的大小。

4. 如果没有足够的空间或者 VM 较大（G、GS 或 E 系列），则可随后更改创建此文件时所在的位置，将该位置指向任何其他附加到 VM 的数据磁盘。 若要更改位置，必须更改以下注册表项：

    ```
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f

    reg unload HKLM\BROKENSYSTEM
    ```

5. [分离 OS 磁盘，然后将 OS 磁盘重新附加到受影响的 VM](./troubleshoot-recovery-disks-portal-windows.md)。

6. 启动 VM 并访问串行控制台。

7. 选择“发送不可屏蔽中断(NMI)”以触发内存转储。

    :::image type="content" source="./media/boot-error-troubleshooting-windows/send-nonmaskable-interrupt.png" alt-text="“正在重启”、“正在关闭”或“正在停止服务”屏幕":::

8. 再次将 OS 磁盘附加到恢复 VM，收集转储文件。

## <a name="contact-azure-support"></a>联系 Azure 支持部门

收集转储文件后，请联系 Azure 支持人员来确定根本原因。

<!-- Update_Description: update meta properties, wording update, update link -->