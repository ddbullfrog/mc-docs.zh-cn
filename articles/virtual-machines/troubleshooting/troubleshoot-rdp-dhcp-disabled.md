---
title: 由于 DHCP 被禁用而无法远程连接到 Azure 虚拟机 | Azure
description: 了解如何排查由于 DHCP 客户端服务在 Azure 中被禁用而导致的 RDP 问题。| Azure
services: virtual-machines-windows
manager: dcscontentpm
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
origin.date: 11/13/2018
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: cbc663de2d6311a975848297339f9b056eca0d27
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104349"
---
# <a name="cannot-rdp-to-azure-virtual-machines-because-the-dhcp-client-service-is-disabled"></a>因 DHCP 客户端服务已禁用而无法通过 RDP 连接到 Azure 虚拟机

本文介绍了在 Azure Windows 虚拟机 (VM) 中禁用 DHCP 客户端服务后无法通过远程桌面连接到该 VM 的问题。

## <a name="symptoms"></a>症状
无法在 Azure 中与 VM 建立 RDP 连接，因为 DHCP 客户端服务在 VM 中被禁用。 在 Azure 门户中的[“启动诊断”](../troubleshooting/boot-diagnostics.md)中检查屏幕截图时，你看到 VM 正常启动并且在登录屏幕中等待凭据。 在使用事件查看器远程查看 VM 中的事件日志时， 发现 DHCP 客户端服务未启动或无法启动。 下面是示例日志：

**日志名称** ：系统 <br />
**源** ：服务控制管理器 <br />
**日期** ：2015/12/16 11:19:36 AM <br />
**事件 ID** ：7022 <br />
**任务类别** ：无 <br />
**级别** ：错误 <br />
**关键字** ：经典<br />
**用户** ：不适用 <br />
**计算机** ：myvm.cosotos.com<br />
**说明** ：DHCP 客户端服务在启动时挂起。<br />

对于资源管理器 VM，可使用串行访问控制台功能，通过以下命令查询事件日志 7022：

```console
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Service Control Manager'] and EventID=7022 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
```

对于经典 VM，你需要在“脱机”模式下工作，并手动收集日志。

## <a name="cause"></a>原因

DHCP 客户端服务未在 VM 上运行。

> [!NOTE]
> 本文仅适用于 DHCP 客户端服务，而不适用于 DHCP 服务器。

## <a name="solution"></a>解决方案

在执行这些步骤之前，请创建受影响 VM 的 OS 磁盘的快照作为备份。 有关详细信息，请参阅[拍摄磁盘快照](../windows/snapshot-copy-managed-disk.md)。

若要解决此问题，请为 VM [重置网络接口](reset-network-interface.md)。

<!-- Not Available on Serial control to enable DHCP or -->
<!-- Not Available on ### Use Serial control-->

### <a name="repair-the-vm-offline"></a>修复 VM 脱机

#### <a name="attach-the-os-disk-to-a-recovery-vm"></a>将 OS 磁盘附加到恢复 VM

1. [将 OS 磁盘附加到恢复 VM](./troubleshoot-recovery-disks-portal-windows.md)。
2. 开始与恢复 VM 建立远程桌面连接。 确保附加的磁盘在磁盘管理控制台中标记为“联机”。  请注意分配给附加的 OS 磁盘的驱动器号。
3. 打开权限提升的命令提示符实例（“以管理员身份运行”）。  然后运行以下脚本。 此脚本假设分配给附加的 OS 磁盘的驱动器号为 **F** 。使用 VM 中的值适当地替换该字母。

    ```
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM

    REM Set default values back on the broken service
    reg add "HKLM\BROKENSYSTEM\ControlSet001\services\DHCP" /v start /t REG_DWORD /d 2 /f
    reg add "HKLM\BROKENSYSTEM\ControlSet001\services\DHCP" /v ObjectName /t REG_SZ /d "NT Authority\LocalService" /f
    reg add "HKLM\BROKENSYSTEM\ControlSet001\services\DHCP" /v type /t REG_DWORD /d 16 /f
    reg add "HKLM\BROKENSYSTEM\ControlSet002\services\DHCP" /v start /t REG_DWORD /d 2 /f
    reg add "HKLM\BROKENSYSTEM\ControlSet002\services\DHCP" /v ObjectName /t REG_SZ /d "NT Authority\LocalService" /f
    reg add "HKLM\BROKENSYSTEM\ControlSet002\services\DHCP" /v type /t REG_DWORD /d 16 /f

    reg unload HKLM\BROKENSYSTEM
    ```

4. [分离 OS 磁盘并重新创建 VM](./troubleshoot-recovery-disks-portal-windows.md)。 然后检查是否解决了问题。

## <a name="next-steps"></a>后续步骤

如果仍需帮助，请[联系支持人员](https://support.azure.cn/support/support-azure/)以快速解决问题。

<!-- Update_Description: update meta properties, wording update, update link -->