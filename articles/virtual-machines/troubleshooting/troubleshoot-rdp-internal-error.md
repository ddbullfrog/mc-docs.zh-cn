---
title: 与 Azure 虚拟机建立 RDP 连接时发生内部错误 | Azure
description: 了解如何排查 Azure 中的 RDP 内部错误 | Azure
services: virtual-machines-windows
manager: dcscontentpm
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
origin.date: 10/22/2018
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 10/19/2020
ms.author: v-yeche
ms.openlocfilehash: 6f2741c48d2ca2f67378784b9cf6dbff730ac177
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104341"
---
# <a name="an-internal-error-occurs-when-you-try-to-connect-to-an-azure-vm-through-remote-desktop"></a>尝试通过远程桌面连接到 Azure VM 时发生内部错误

本文介绍了尝试连接到 Azure 中的虚拟机 (VM) 时可能会遇到的错误。

## <a name="symptoms"></a>症状

无法使用远程桌面协议 (RDP) 连接到 Azure VM。 连接过程停滞在“正在配置远程连接”阶段，或收到以下错误消息：

- RDP 内部错误
- 发生了内部错误
- 此计算机无法连接到远程计算机。 请再次尝试连接。 如果问题持续出现，请与远程计算机的所有者或网络管理员联系。

## <a name="cause"></a>原因

此问题可能是以下原因造成的：

- 虚拟机可能已遭到攻击。
- 无法访问本地 RSA 加密密钥。
- 已禁用 TLS 协议。
- 证书已损坏或过期。

## <a name="solution"></a>解决方案

在执行这些步骤之前，请创建受影响 VM 的 OS 磁盘的快照作为备份。 有关详细信息，请参阅[拍摄磁盘快照](../windows/snapshot-copy-managed-disk.md)。

若要排查此问题，可通过将 VM 的 OS 磁盘附加到恢复 VM 来[修复 VM 脱机](#repair-the-vm-offline)。

<!--Not Available on use the Serial Console or-->
<!-- Not Available on ### Use Serial control-->
### <a name="repair-the-vm-offline"></a>修复 VM 脱机

#### <a name="attach-the-os-disk-to-a-recovery-vm"></a>将 OS 磁盘附加到恢复 VM

1. [将 OS 磁盘附加到恢复 VM](./troubleshoot-recovery-disks-portal-windows.md)。
2. 将 OS 磁盘附加到恢复 VM 后，请确保磁盘在磁盘管理控制台中标记为“联机”  。 请注意分配给附加的 OS 磁盘的驱动器号。
3. 开始与恢复 VM 建立远程桌面连接。

#### <a name="enable-dump-log-and-serial-console"></a>启用转储日志和串行控制台

若要启用转储日志和串行控制台，请运行以下脚本。

1. 打开权限提升的命令提示符会话（“以管理员身份运行”）。 
2. 运行以下脚本：

    对于此脚本，我们假设分配给附加 OS 磁盘的驱动器号为 F。请将此驱动器号替换为 VM 中的相应值。

    ```console
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
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    reg unload HKLM\BROKENSYSTEM
    ```

#### <a name="reset-the-permission-for-machinekeys-folder"></a>重置 MachineKeys 文件夹的权限

1. 打开权限提升的命令提示符会话（“以管理员身份运行”）。 
2. 运行以下脚本。 对于此脚本，我们假设分配给附加 OS 磁盘的驱动器号为 F。请将此驱动器号替换为 VM 中的相应值。

    ```console
    Md F:\temp

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\BeforeScript_permissions.txt

    takeown /f "F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys" /a /r

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\System:(F)"

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\NETWORK SERVICE:(R)"

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "BUILTIN\Administrators:(F)"

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\AfterScript_permissions.txt
    ```

#### <a name="enable-all-supported-tls-versions"></a>启用所有受支持的 TLS 版本

1. 打开权限提升的命令提示符会话（“以管理员身份运行”），然后运行以下命令。  以下脚本假设分配给附加 OS 磁盘的驱动器号为 F。请将此驱动器号替换为 VM 中的相应值。
2. 检查启用了哪个 TLS：

    ```console
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWO
    ```

3. 如果该密钥不存在或者其值为 **0** ，请运行以下脚本来启用该协议：

    ```console
    REM Enable TLS 1.0, TLS 1.1 and TLS 1.2

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f
    ```

4. 启用 NLA：

    ```console
    REM Enable NLA

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\Terminal Server\WinStations\RDP-Tcp" /v fAllowSecProtocolNegotiation /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\Terminal Server\WinStations\RDP-Tcp" /v fAllowSecProtocolNegotiation /t REG_DWORD /d 1 /f reg unload HKLM\BROKENSYSTEM
    ```

5. [拆离 OS 磁盘并重新创建 VM](./troubleshoot-recovery-disks-portal-windows.md)，然后检查问题是否得以解决。

<!-- Update_Description: update meta properties, wording update, update link -->