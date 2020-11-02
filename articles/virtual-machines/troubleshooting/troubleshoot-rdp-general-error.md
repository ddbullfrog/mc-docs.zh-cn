---
title: 排查通过 RDP 连接到 Azure 中的 Windows VM 时发生的常规错误 | Azure
description: 了解如何排查通过 RDP 连接到 Azure 中的 Windows VM 时发生的常规错误 | Azure
services: virtual-machines-windows
manager: dcscontentpm
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
origin.date: 10/31/2018
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: 39d9c56a854a4e920f182ae5de480c86bc91a322
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104344"
---
# <a name="troubleshoot-an-rdp-general-error-in-azure-vm"></a>排查 Azure VM 的常规 RDP 错误

本文描述在与 Azure 中的 Windows 虚拟机 (VM) 建立远程桌面协议 (RDP) 连接时可能遇到的常规错误。

## <a name="symptom"></a>症状

与 Azure 中的 Windows VM 建立 RDP 连接时，可能会收到以下常规错误消息：

**由于以下原因之一，远程桌面无法连接到远程计算机：**

1. **未启用服务器的远程访问**

2. **远程计算机已关闭**

3. **远程计算机在网络中不可用**

**请确保远程计算机已打开并已连接到网络，并且已启用远程访问。**

## <a name="cause"></a>原因

此问题可能由以下原因导致：

### <a name="cause-1"></a>原因 1

RDP 组件已在如下所述的位置禁用：

- 组件级别
- 侦听器级别
- 终端服务器
- 远程桌面会话主机角色

### <a name="cause-2"></a>原因 2

远程桌面服务 (TermService) 未运行。

### <a name="cause-3"></a>原因 3

RDP 侦听器配置不当。

## <a name="solution"></a>解决方案

在执行这些步骤之前，请创建受影响 VM 的 OS 磁盘的快照作为备份。 要解决此问题，请脱机修复 VM。

<!--Not Available on use Serial control or-->
<!-- Not Available on ### Serial Console-->

### <a name="offline-repair"></a>脱机修复

#### <a name="step-1-turn-on-remote-desktop"></a>步骤 1：启用远程桌面

1. [将 OS 磁盘附加到恢复 VM](./troubleshoot-recovery-disks-portal-windows.md)。
2. 开始与恢复 VM 建立远程桌面连接。
3. 确保磁盘在磁盘管理控制台中标记为“联机”。 请注意分配给附加的 OS 磁盘的驱动器号。
4. 开始与恢复 VM 建立远程桌面连接。
5. 打开权限提升的命令提示符会话（“以管理员身份运行”）。 运行以下脚本。 对于此脚本，我们假设分配给附加 OS 磁盘的驱动器号为 F。请将此驱动器号替换为 VM 中的相应值。

    ```
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv 
    reg load HKLM\BROKENSOFTWARE F:\windows\system32\config\SOFTWARE.hiv 

    REM Ensure that Terminal Server is enabled 

    reg add "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server" /v TSEnabled /t REG_DWORD /d 1 /f 
    reg add "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server" /v TSEnabled /t REG_DWORD /d 1 /f 

    REM Ensure Terminal Service is not set to Drain mode 
    reg add "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server" /v TSServerDrainMode /t REG_DWORD /d 0 /f 
    reg add "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server" /v TSServerDrainMode /t REG_DWORD /d 0 /f 

    REM Ensure Terminal Service has logon enabled 
    reg add "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server" /v TSUserEnabled /t REG_DWORD /d 0 /f 
    reg add "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server" /v TSUserEnabled /t REG_DWORD /d 0 /f 

    REM Ensure the RDP Listener is not disabled 
    reg add "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v fEnableWinStation /t REG_DWORD /d 1 /f 
    reg add "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v fEnableWinStation /t REG_DWORD /d 1 /f 

    REM Ensure the RDP Listener accepts logons 
    reg add "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v fLogonDisabled /t REG_DWORD /d 0 /f 
    reg add "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v fLogonDisabled /t REG_DWORD /d 0 /f 

    REM RDP component is enabled 
    reg add "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f 
    reg add "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f 
    reg add "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fDenyTSConnections /t REG_DWORD /d 0 /f 

    reg unload HKLM\BROKENSYSTEM 
    reg unload HKLM\BROKENSOFTWARE 
    ```

6. 如果 VM 已加入域，请检查以下注册表项，以查看是否有某个组策略禁用了 RDP。 

    ```
    HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services\fDenyTSConnectionS
    ```

    如果此注册表项的值设置为 1，则表示策略禁用了 RDP。 若要通过 GPO 策略启用远程桌面，请在域控制器中更改以下策略：

    **计算机配置\策略\管理模板：**

    策略定义\Windows 组件\远程桌面服务\远程桌面会话主机\连接\允许用户使用远程桌面服务进行远程连接

1. 从救援 VM 拆离磁盘。
1. [从磁盘创建新的 VM](../windows/create-vm-specialized.md)。

如果仍发生此问题，请转到步骤 2。

#### <a name="step-2-enable-remote-desktop-services"></a>步骤 2：启用远程桌面服务

有关详细信息，请参阅[远程桌面服务在 Azure VM 上不启动](troubleshoot-remote-desktop-services-issues.md)。

#### <a name="step-3-reset-rdp-listener"></a>步骤 3：重置 RDP 侦听器

有关详细信息，请参阅[远程桌面经常在 Azure VM 中断开连接](troubleshoot-rdp-intermittent-connectivity.md)。

## <a name="need-help-contact-support"></a>需要帮助？ 联系支持人员

如果仍需要帮助，可 [联系支持人员](https://support.azure.cn/support/support-azure/) 来快速解决问题。

<!-- Update_Description: update meta properties, wording update, update link -->