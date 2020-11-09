---
title: Windows 停止错误 状态系统进程已终止
description: 本文提供了一些步骤，用于解决操作系统遇到停止错误 0xC000021A，从而阻止 Azure 虚拟机启动的问题。
services: virtual-machines-windows, azure-resource-manager
manager: dcscontentpm
tags: azure-resource-manager
ms.assetid: 037d0858-4611-4707-bd46-cc8085d011ed
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
origin.date: 09/21/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: no
ms.testdate: 10/19/2020
ms.author: v-yeche
ms.openlocfilehash: 98e06689117724f838516bc9d9af5c7b9fea8ab0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105465"
---
<!--Verified Successfully-->
# <a name="windows-stop-error---0xc000021a-status-system-process-terminated"></a>Windows 停止错误 - 0xC000021A 状态系统进程终止

本文提供了一些步骤，用于解决操作系统 (OS) 遇到停止错误 0xC000021A，从而使 Azure 虚拟机 (VM) 无法启动的问题。

## <a name="symptom"></a>症状

使用[启动诊断](./boot-diagnostics.md)查看 VM 的屏幕截图时，屏幕截图将显示 OS 在启动过程中遇到错误的消息，并显示以下消息：

你的电脑遇到问题，需要重启 **。我们将收集一些错误信息，然后你就可以重启。（已完成 ##%）如果想了解更多信息，可以稍后联机搜索以下错误：0xC000021a** 。

![图 1 显示了错误代码 #0xC000021A 和消息“你的电脑遇到问题，需要重启。 我们将收集一些错误信息，然后你就可以重启。”](./media/windows-stop-error-system-process-terminated/1-pc-problem-restart.png)

## <a name="cause"></a>原因

错误 0xC000021A 表示“STATUS_SYSTEM_PROCESS_TERMINATED”。

当关键进程（如 WinLogon (winlogon.exe) 或客户端服务器运行时子系统 (csrss.exe)）失败时，会发生此错误。 一旦内核检测到这些服务中的任一服务已停止，就会引发“STOP 0xC000021A”错误。 此错误可能有多种原因，包括：

- 安装了不匹配的系统文件。
- Service Pack 或 KB 更新安装失败。
- 用于还原硬盘的备份程序未正确还原可能已在使用的文件。
- 安装了不兼容的第三方程序。

## <a name="solution"></a>解决方案

### <a name="collect-the-memory-dump-file"></a>收集内存转储文件

若要解决此问题，需要对故障转储进行分析。 收集故障的内存转储文件，并联系支持人员。 若要收集转储文件，请执行以下步骤：

### <a name="attach-the-os-disk-to-a-new-repair-vm"></a>将 OS 磁盘附加到新的修复 VM

1. 使用 [VM 修复命令](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) 的步骤 1-3 准备一个修复 VM。
2. 使用“远程桌面连接”来连接到修复 VM。

### <a name="locate-the-dump-file-and-submit-a-support-ticket"></a>找到转储文件并提交支持票证

1. 在修复 VM 上，转到附加的 OS 磁盘中的 Windows 文件夹。 如果分配给附加 OS 磁盘的驱动器号为 F，请转到 F:\Windows。
2. 找到 memory.dmp 文件，然后使用该内存转储文件[提交支持票证](https://support.azure.cn/support/support-azure/)。
3. 如果在查找 memory.dmp 文件时遇到问题，可以按照指南[使用 NMI 调用来生成故障转储文件](https://docs.microsoft.com/windows/client-management/generate-kernel-or-complete-crash-dump)。

    <!--Not Available on [non-maskable interrupt (NMI) calls in serial console](./serial-console-windows.md#use-the-serial-console-for-nmi-calls)-->

## <a name="next-steps"></a>后续步骤

- 有关故障排除的详细信息，请参阅[排查常见启动错误](./boot-error-troubleshoot.md)或[通过将 OS 磁盘附加到恢复 VM 对 Windows VM 进行故障排除](./troubleshoot-recovery-disks-windows.md)。 还应熟悉[如何使用启动诊断对虚拟机进行故障排除](./boot-diagnostics.md)。
- 有关使用 Resource Manager 的详细信息，请参阅 [Azure Resource Manager 概述](../../azure-resource-manager/management/overview.md)。
- 如果无法连接到 VM，请参阅[对 Azure VM 的 RDP 连接进行故障排除](./troubleshoot-rdp-connection.md)。

<!-- Update_Description: update meta properties, wording update, update link -->