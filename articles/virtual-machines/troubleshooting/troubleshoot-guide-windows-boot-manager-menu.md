---
title: Windows 虚拟机因 Windows 启动管理器而无法启动
description: 本文提供了当 Windows 启动管理器阻止 Azure 虚拟机启动时用于解决问题的步骤。
services: virtual-machines-windows
manager: dcscontentpm
tags: azure-resource-manager
ms.assetid: a97393c3-351d-4324-867d-9329e31b3598
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
origin.date: 03/26/2020
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: c0b78e1b6b9292dd1f26a1111d534eb59aeb8646
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106324"
---
<!--Verified successfully-->
# <a name="windows-vm-cannot-boot-due-to-windows-boot-manager"></a>Windows VM 因 Windows 启动管理器而无法启动

本文提供了相关步骤，用于解决 Windows 启动管理器阻止 Azure 虚拟机 (VM) 启动的问题。

## <a name="symptom"></a>症状

VM 一直在等待用户提示。除非用户手动进行指示，否则 VM 不会启动。

使用[启动诊断](./boot-diagnostics.md)查看 VM 的屏幕截图时，会看到屏幕截图显示 Windows 启动管理器，其中包含消息“请选择要启动的操作系统，或按 TAB 键选择一项工具:”  。

图 1

![Windows 启动管理器，其中包含消息“请选择要启动的操作系统，或按 TAB 键选择一项工具:”](media/troubleshoot-guide-windows-boot-manager-menu/1.jpg)

## <a name="cause"></a>原因

此错误是由 Windows 启动管理器中的 BCD 标志“displaybootmenu”  导致的。 启用该标志后，Windows 启动管理器会在启动过程中提示用户选择要运行的加载程序，导致启动延迟。 在 Azure 中，此功能可能会增加启动 VM 所需的时间。

## <a name="solution"></a>解决方案

过程概述：

<!--Not Available on 1. Configure for Faster Boot Time using Serial Console.-->

1. 创建并访问修复 VM。
2. 在修复 VM 上进行配置以缩短启动时间。
3. **建议** ：在重建 VM 之前，启用串行控制台和内存转储收集。
4. 重建 VM。

<!--Not Available on ### Configure for Faster Boot Time using Serial Console-->

### <a name="create-and-access-a-repair-vm"></a>创建并访问一个修复 VM

1. 使用 [VM 修复命令的步骤 1-3](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) 来准备一个修复 VM。
2. 使用远程桌面连接来连接到修复 VM。

### <a name="configure-for-faster-boot-time-on-a-repair-vm"></a>在修复 VM 上进行配置以缩短启动时间。

1. 打开权限提升的命令提示符。
2. 输入以下命令来启用 DisplayBootMenu：

    对于第 1 代 VM  ，请使用以下命令：

    `bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /set {bootmgr} displaybootmenu yes`

    对于第 2 代 VM  ，请使用以下命令：

    `bcdedit /store <VOLUME LETTER OF EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /set {bootmgr} displaybootmenu yes`

    替换所有大于号或小于号以及其中的文本，例如“< text here >”。

3. 将超时值更改为 5 秒：

    对于第 1 代 VM  ，请使用以下命令：

    `bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /set {bootmgr} timeout 5`

    对于第 2 代 VM  ，请使用以下命令：

    `bcdedit /store <VOLUME LETTER OF EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /set {bootmgr} timeout 5`

    替换所有大于号或小于号以及其中的文本，例如“< text here >”。

### <a name="recommended-before-you-rebuild-the-vm-enable-serial-console-and-memory-dump-collection"></a>建议：在重建 VM 之前，启用串行控制台和内存转储收集

若要启用内存转储收集和串行控制台，请运行以下脚本：

1. 打开提升的命令提示符会话（以管理员身份运行）。
2. 运行以下命令：

    启用串行控制台

    `bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON`

    `bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200`

    替换所有大于号或小于号以及其中的文本，例如“< text here >”。

3. 验证 OS 磁盘上的可用空间是否与 VM 上的内存大小 (RAM) 相当。

    如果 OS 磁盘上没有足够的空间，则应更改内存转储文件的创建位置，并将该位置指向 VM 上附加的具有足够可用空间的任何数据磁盘。 若要更改位置，请将以下命令中的“%SystemRoot%”替换为数据磁盘的驱动器号（例如“F:”）。

#### <a name="suggested-configuration-to-enable-os-dump"></a>用于启用 OS 转储的建议配置

**加载损坏的 OS 磁盘：**

`REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM`

**在 ControlSet001 上启用：**

`REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f`

`REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f`

`REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f`

**在 ControlSet002 上启用：**

`REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f`

`REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f`

`REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f`

**卸载损坏的 OS 磁盘：**

`REG UNLOAD HKLM\BROKENSYSTEM`

### <a name="rebuild-the-original-vm"></a>重建原始 VM

使用 [VM 修复命令的步骤 5](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) 重新装配 VM。

<!-- Update_Description: update meta properties, wording update, update link -->