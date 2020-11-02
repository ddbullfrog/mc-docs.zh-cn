---
title: Azure VM 中远程桌面频繁断开连接 | Azure
description: 了解如何解决 Azure VM 中远程桌面频繁断开连接的问题。
services: virtual-machines-windows
manager: dcscontentpm
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
origin.date: 10/24/2018
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: 6117b5929841f1e509d231447bb4980c53688c55
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104342"
---
# <a name="remote-desktop-disconnects-frequently-in-azure-vm"></a>Azure VM 中远程桌面频繁断开连接

本文介绍如何解决通过远程桌面协议 (RDP) 连接到 Azure 虚拟机 (VM) 频繁断开的问题。

## <a name="symptom"></a>症状

会话过程中遇到间歇性的 RDP 连接问题。 最初可以连接到 VM，但随后连接中断。

## <a name="cause"></a>原因

如果 RDP 侦听器配置不当，可能会出现此问题。 通常情况下，在使用自定义映像的 VM 上会发生此问题。

## <a name="solution"></a>解决方案

在执行这些步骤之前，[创建受影响 VM 的 OS 磁盘的快照](../windows/snapshot-copy-managed-disk.md)作为备份。 

若要排查此问题，可通过将 VM 的 OS 磁盘附加到恢复 VM 来[修复 VM 脱机](#repair-the-vm-offline)。

<!-- Not Available on ### Serial control-->

### <a name="repair-the-vm-offline"></a>修复 VM 脱机

1. [将 OS 磁盘附加到恢复 VM](./troubleshoot-recovery-disks-portal-windows.md)。
2. 将 OS 磁盘附加到恢复 VM 后，请确保磁盘在磁盘管理控制台中标记为“联机”  。 请注意分配给附加的 OS 磁盘的驱动器号。
3. 在附加的 OS 磁盘上，浏览到 \windows\system32\config 文件夹  。 将此文件夹中的所有文件复制为备份，以备回滚之需。
4. 启动注册表编辑器 (regedit.exe)。
5. 选择“HKEY_LOCAL_MACHINE”  项。 在菜单上，选择“文件”   > “加载配置单元”  ：
6. 浏览到已附加 OS 磁盘上的 **\windows\system32\config\SYSTEM** 文件夹。 输入“BROKENSYSTEM”  作为配置单元名称。 新的注册表配置单元将显示在“HKEY_LOCAL_MACHINE”  项之下。 然后加载 HKEY_LOCAL_MACHINE 密钥下的软件配置单元 \windows\system32\config\SOFTWARE   。 输入“BROKENSOFTWARE”  作为配置单元软件。 
7. 打开提升的命令提示符窗口（以管理员身份运行），然后在其余步骤中运行命令以重置 RDP 配置  。 
8. 将 RDP 的安全层级降为 0，以便服务器和客户端之间的通信使用本机 RDP 加密：

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'SecurityLayer' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'SecurityLayer' /t REG_DWORD /d 0 /f`

9. 将加密级别降低到最小设置，以允许旧的 RDP 客户端进行连接：

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'MinEncryptionLevel' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'MinEncryptionLevel' /t REG_DWORD /d 1 /f`

10. 将 RDP 设置为加载客户端计算机的用户配置。

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fQueryUserConfigFromLocalMachine' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fQueryUserConfigFromLocalMachine' /t REG_DWORD /d 1 /f`

11. 启用“RDP 保持连接”控制：

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'KeepAliveTimeout' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'KeepAliveTimeout' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'KeepAliveEnable' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'KeepAliveInterval' /t REG_DWORD /d 1 /f`

12. 设置“RDP 重新连接”控制：

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritReconnectSame' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fReconnectSame' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritReconnectSame' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fReconnectSame' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'fDisableAutoReconnect' /t REG_DWORD /d 0 /f`

13. 设置“RDP 会话时间”控制：

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxSessionTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxSessionTime' /t REG_DWORD /d 1 /f`

14. 设置“RDP 断开连接时间”控制：

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxDisconnectionTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxDisconnectionTime' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxDisconnectionTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxDisconnectionTime' /t REG_DWORD /d 0 /f`

15. 设置“RDP 连接时间”控制：

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxConnectionTime' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxConnectionTime' /t REG_DWORD /d 0 /f`

16. 设置“RDP 会话空闲时间”控制：

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxIdleTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v ' MaxIdleTime' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxIdleTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v ' MaxIdleTime' /t REG_DWORD /d 0 /f`

17. 设置“限制最大并发连接数”控制：

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxInstanceCount' /t REG_DWORD /d ffffffff /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxInstanceCount' /t REG_DWORD /d ffffffff /f`

18. 重启 VM，并再次尝试使用 RDP 连接到它。

## <a name="need-help"></a>需要帮助？ 
请联系支持人员。 如果仍需要帮助，可 [联系支持人员](https://support.azure.cn/support/support-azure/) 来快速解决问题。

<!-- Update_Description: update meta properties, wording update, update link -->