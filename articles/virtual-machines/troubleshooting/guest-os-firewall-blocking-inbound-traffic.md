---
title: Azure VM 来宾 OS 防火墙阻止入站流量 | Azure
description: 了解如何修复来宾操作系统防火墙阻止入站流量的远程桌面门户 (RDP) 连接问题。
services: virtual-machines-windows
author: rockboyfor
manager: digimobile
tags: ''
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: azurecli
origin.date: 11/22/2018
ms.date: 04/27/2020
ms.author: v-yeche
ms.openlocfilehash: 02ea584144f418402955560ee7107dd64baf0354
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105948"
---
# <a name="azure-vm-guest-os-firewall-is-blocking-inbound-traffic"></a>Azure VM 来宾 OS 防火墙阻止入站流量

本文介绍如何修复来宾操作系统防火墙阻止入站流量时出现的远程桌面门户 (RDP) 问题。

## <a name="symptoms"></a>症状

无法使用 RDP 连接来连接 Azure 虚拟机 (VM)。 从“启动诊断”->“屏幕截图”中，可看到操作系统已在欢迎屏幕 (Ctrl+Alt+Del) 上完全加载。

## <a name="cause"></a>原因

### <a name="cause-1"></a>原因 1

未设置 RDP 规则来允许 RDP 流量。

### <a name="cause-2"></a>原因 2

来宾系统防火墙配置文件设置为阻止所有入站连接，包括 RDP 流量。

![防火墙设置](./media/guest-os-firewall-blocking-inbound-traffic/firewall-advanced-setting.png)

## <a name="solution"></a>解决方案

在执行这些步骤之前，请创建受影响 VM 的系统磁盘快照作为备份。 有关详细信息，请参阅[拍摄磁盘快照](../windows/snapshot-copy-managed-disk.md)。

要解决此问题，请使用[如何使用远程工具解决 Azure VM 问题](remote-tools-troubleshoot-azure-vm-issues.md)中介绍的方法远程连接到 VM，然后将来宾操作系统防火墙规则编辑为“允许”RDP 流量  。

<!-- Not Available on ### Online troubleshooting -->
<!-- Not Available on Serial Control-->

### <a name="offline-mitigations"></a>脱机缓解措施

1. [将系统磁盘附加到恢复 VM](troubleshoot-recovery-disks-portal-windows.md)。

2. 开始与恢复 VM 建立远程桌面连接。

3. 确保磁盘在磁盘管理控制台中标记为“联机”。  请留意分配给附加系统磁盘的驱动器号。

#### <a name="mitigation-1"></a>缓解措施 1

请参阅[如何在来宾 OS 上启用/禁用防火墙规则](enable-disable-firewall-rule-guest-os.md)。

#### <a name="mitigation-2"></a>缓解措施 2

1. [将系统磁盘附加到恢复 VM](troubleshoot-recovery-disks-portal-windows.md)。

2. 开始与恢复 VM 建立远程桌面连接。

3. 将系统磁盘附加到恢复 VM 后，请确保磁盘在磁盘管理控制台中标记为“联机”  。 请注意分配给附加的 OS 磁盘的驱动器号。

4. 打开提升后的 CMD 实例，然后运行以下脚本：

    ```cmd
    REM Backup the registry prior doing any change
    robocopy f:\windows\system32\config f:\windows\system32\config.BACK /MT

    REM Mount the hive
    reg load HKLM\BROKENSYSTEM f:\windows\system32\config\SYSTEM

    REM Delete the keys to block all inbound connection scenario
    REG DELETE "HKLM\BROKENSYSTEM\ControlSet001\services\SharedAccess\Parameters\FirewallPolicy\DomainProfile" /v DoNotAllowExceptions
    REG DELETE "HKLM\BROKENSYSTEM\ControlSet001\services\SharedAccess\Parameters\FirewallPolicy\PublicProfile" /v DoNotAllowExceptions
    REG DELETE "HKLM\BROKENSYSTEM\ControlSet001\services\SharedAccess\Parameters\FirewallPolicy\StandardProfile" /v DoNotAllowExceptions
    REG DELETE "HKLM\BROKENSYSTEM\ControlSet002\services\SharedAccess\Parameters\FirewallPolicy\DomainProfile" /v DoNotAllowExceptions
    REG DELETE "HKLM\BROKENSYSTEM\ControlSet002\services\SharedAccess\Parameters\FirewallPolicy\PublicProfile" /v DoNotAllowExceptions
    REG DELETE "HKLM\BROKENSYSTEM\ControlSet002\services\SharedAccess\Parameters\FirewallPolicy\StandardProfile" /v DoNotAllowExceptions

    REM Unmount the hive
    reg unload HKLM\BROKENSYSTEM
    ```

5. [拆离系统磁盘并重新创建 VM](troubleshoot-recovery-disks-portal-windows.md)。

6. 检查是否解决了问题。

<!-- Update_Description: update link -->
