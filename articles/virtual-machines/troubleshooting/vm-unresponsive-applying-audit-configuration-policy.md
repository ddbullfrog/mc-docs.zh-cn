---
title: 在应用审核策略配置策略时，虚拟机无响应
description: 本文提供了相关步骤，用于解决在应用审核策略配置策略时，虚拟机无响应，从而阻止 Azure VM 启动的问题。
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 3f6383b5-81fa-49ea-9434-2fe475e4cbef
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
origin.date: 08/24/2020
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 4847fda0e991e45748427d13caf93dc259b6a439
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92127834"
---
<!--Verified successfully on Charactors only-->
# <a name="virtual-machine-is-unresponsive-while-applying-audit-policy-configuration-policy"></a>在应用审核策略配置策略时，虚拟机无响应

本文提供了相关步骤，用于解决在应用审核策略配置策略时，虚拟机无响应，从而阻止 Azure VM 启动的问题。

## <a name="symptom"></a>症状

使用[启动诊断](/virtual-machines/troubleshooting/boot-diagnostics)查看 VM 的屏幕截图时，将看到屏幕截图显示操作系统 (OS) 在启动过程中无响应，并显示消息“应用审核策略配置策略”。

  ![OS 启动时将显示以下消息：“正在应用审核策略配置策略”](./media/vm-unresponsive-applying-audit-configuration-policy/1.png)

  ![OS 在 Windows Server 2012 中启动时将显示以下消息：“正在应用审核策略配置策略”](./media/vm-unresponsive-applying-audit-configuration-policy/2.png)

## <a name="cause"></a>原因

当策略尝试清除旧的用户配置文件时，会发生锁定冲突。

> [!NOTE]
> 这仅针对于 Windows Server 2012 和 Windows Server 2012 R2。

下面是有问题的策略：“计算机配置”\“策略”\“管理模板”\“系统/用户配置文件”\“在系统重启时，删除超过指定天数的用户配置文件”。

## <a name="solution"></a>解决方案

### <a name="process-overview"></a>过程概述

1. 创建和访问修复 VM。
1. 禁用策略。
1. 启用串行控制台和内存转储收集。
1. 重新生成 VM。
1. 收集内存转储文件并提交支持工单。

### <a name="create-and-access-a-repair-vm"></a>创建和访问修复 VM

1. 使用 [VM 修复命令](/virtual-machines/troubleshooting/repair-windows-vm-using-azure-virtual-machine-repair-commands)的步骤 1-3 来准备一个修复 VM。
1. 使用远程桌面连接来连接到修复 VM。

### <a name="disable-the-policy"></a>禁用策略

1. 在修复 VM 上，打开“注册表编辑器”。
1. 找到“HKEY_LOCAL_MACHINE”项，然后从菜单中选择“文件”>“加载配置单元” 。

    :::image type="content" source="./media/vm-unresponsive-applying-audit-configuration-policy/3.png" alt-text="注册表编辑器中用于加载配置单元的导航。":::

    - 可以使用加载配置单元从脱机系统加载注册表项。 在这种情况下，系统是附加到修复 VM 的受损磁盘。
    - 系统范围内的设置存储在 HKEY_LOCAL_MACHINE 上，可以缩写为 HKLM 。

1. 在附加的磁盘中，打开 `\windows\system32\config\SOFTWARE` 文件。

    - 当系统提示你输入名称时，请输入 BROKENSOFTWARE。
    - 若要验证是否已加载 BROKENSOFTWARE，请展开“HKEY_LOCAL_MACHINE”并查找已添加的 BROKENSOFTWARE 项  。

1. 转到“BROKENSOFTWARE”，并检查加载的配置单元中是否有“CleanupProfile”项 。

    - 如果该项存在，说明已设置 CleanupProfile 策略。 它的值表示以天为单位的保留策略。
    - 如果该项不存在，说明未设置 CleanupProfile 策略。 在这种情况下，请跳到[连同内存转储文件一起提交支持工单](#collect-the-memory-dump-file-and-submit-a-support-ticket)。

1. 使用以下命令删除 CleanupProfiles 项：

    `reg delete &quot;HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows\System" /v CleanupProfiles /f`

1. 使用以下命令卸载 BROKENSOFTWARE 配置单元：

    `reg unload HKLM\BROKENSOFTWARE`

### <a name="enable-the-serial-console-and-memory-dump-collection"></a>启用串行控制台和内存转储收集

**建议** ：在重新生成 VM 之前，通过运行以下脚本来启用串行控制台和内存转储收集：

1. 以管理员身份打开权限提升的命令提示符会话。
1. 列出 BCD 存储数据，并确定要在下一步中使用的引导加载程序标识符。

    1. 对于第 1 代 VM，请输入以下命令，并记下列出的标识符：

        `bcdedit /store <BOOT PARTITON>:\boot\bcd /enum`

        - 在该命令中，将 `<BOOT PARTITON>` 替换为附加磁盘中包含引导文件夹的分区驱动器号。

            :::image type="content" source="./media/vm-unresponsive-applying-audit-configuration-policy/4.png" alt-text="注册表编辑器中用于加载配置单元的导航。":::

    - 可以使用加载配置单元从脱机系统加载注册表项。 在这种情况下，系统是附加到修复 VM 的受损磁盘。
    - 系统范围内的设置存储在 HKEY_LOCAL_MACHINE 上，可以缩写为 HKLM 。

1. 在附加的磁盘中，打开 `\windows\system32\config\SOFTWARE` 文件。

    - 当系统提示你输入名称时，请输入 BROKENSOFTWARE。
    - 若要验证是否已加载 BROKENSOFTWARE，请展开“HKEY_LOCAL_MACHINE”并查找已添加的 BROKENSOFTWARE 项  。

1. 转到“BROKENSOFTWARE”，并检查加载的配置单元中是否有“CleanupProfile”项 。

    - 如果该项存在，说明已设置 CleanupProfile 策略。 它的值表示以天为单位的保留策略。
    - 如果该项不存在，说明未设置 CleanupProfile 策略。 在这种情况下，请跳到[连同内存转储文件一起提交支持工单](#collect-the-memory-dump-file-and-submit-a-support-ticket)。

1. 使用以下命令删除 CleanupProfiles 项：

    `reg delete &quot;HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows\System" /v NMICrashDump /t REG_DWORD /d 1 /f 
    ```

    **卸载损坏的 OS 磁盘：**

    ```
    REG UNLOAD HKLM\BROKENSYSTEM
    ```

### <a name="rebuild-the-virtual-machine"></a>重新生成虚拟机

1. 使用 [VM 修复命令的步骤 5](/virtual-machines/troubleshooting/repair-windows-vm-using-azure-virtual-machine-repair-commands#repair-process-example) 重新生成 VM。

1. 测试 VM 是否正常启动以查看问题是否已解决。

   - 如果问题尚未解决，请继续转到[收集转储文件并提交支持工单](#collect-the-memory-dump-file-and-submit-a-support-ticket)。
   - 如果问题已解决，则无需执行其他步骤。

如果问题得到解决，说明已在本地禁用策略。 对于永久性解决方案，请勿在 VM 上使用 CleanupProfiles 策略，因为它将自动删除用户配置文件。 使用其他方法执行配置文件清理，例如计划任务或脚本。

请勿使用此策略：
“计算机”\“管理模板”\“系统”/“用户配置文件”\“在系统重启时，删除超过指定天数的用户配置文件”。

### <a name="the-issue-should-now-be-fixed"></a>此问题现在应已修复

测试 VM，确保其正常运行。 如果仍遇到问题，可以继续阅读下一节以获得进一步的帮助。

### <a name="collect-the-memory-dump-file-and-submit-a-support-ticket"></a>收集内存转储文件并提交支持工单

若要解决此问题，需先收集故障的内存转储文件，然后使用此内存转储文件联系支持部门。 若要收集转储文件，请执行以下步骤：

#### <a name="attach-the-os-disk-to-a-new-repair-vm"></a>将 OS 磁盘附加到新的修复 VM

1. 使用 [VM 修复命令的步骤 1-3](/virtual-machines/troubleshooting/repair-windows-vm-using-azure-virtual-machine-repair-commands) 来准备一个新的修复 VM。
1. 使用远程桌面连接来连接到修复 VM。

#### <a name="locate-the-dump-file-and-submit-a-support-ticket"></a>找到转储文件并提交支持票证

1. 在修复 VM 上，转到附加的 OS 磁盘中的 Windows 文件夹。 如果分配给附加 OS 磁盘的驱动器号标记为 F，则需转到 `F:\Windows`。
1. 找到 `memory.dmp` 文件，然后连同内存转储文件一起[提交支持票证](https://support.azure.cn/support/support-azure/)。

<!--Not Available on [non-maskable interrupt (NMI) calls in serial console](/virtual-machines/troubleshooting/serial-console-windows#use-the-serial-console-for-nmi-calls)-->

<!-- Update_Description: update meta properties, wording update, update link -->
