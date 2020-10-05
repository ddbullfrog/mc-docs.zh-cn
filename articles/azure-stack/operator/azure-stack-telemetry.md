---
title: 配置 Azure Stack Hub 遥测
titleSuffix: Azure Stack
description: 了解 Azure Stack Hub 遥测，以及如何使用 PowerShell 配置遥测设置。
author: WenJason
ms.service: azure-stack
ms.topic: conceptual
origin.date: 1/16/2020
ms.date: 10/12/2020
ms.author: v-jay
ms.reviewer: comartin
ms.lastreviewed: 10/15/2019
ms.openlocfilehash: c6f6ef42628806a97a50afd7ff1e00f4d8cb432e
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91437724"
---
# <a name="configure-azure-stack-hub-telemetry"></a>配置 Azure Stack Hub 遥测

Azure Stack Hub 遥测通过互连用户体验将系统数据自动上传到 Azure。 Azure 团队使用 Azure Stack Hub 遥测收集的数据来改进客户体验。 此数据也用于安全性、运行状况、质量和性能分析。

遥测可为 Azure Stack Hub 操作员提供宝贵的见解来让他们洞察企业部署，并提供有助于构思 Azure Stack Hub 新版本的看法。

Azure Stack Hub 遥测基于 Windows Server 2016 互连用户体验与遥测组件。 此组件使用 [Windows 事件跟踪 (ETW)](https://docs.microsoft.com/windows/win32/tracelogging/trace-logging-about) 跟踪日志记录技术来收集和存储遥测事件与数据。 Azure Stack 组件使用相同的技术，发布使用公共操作系统事件日志记录和跟踪 API 收集的事件与数据。 这些 Azure Stack Hub 组件的示例包括以下提供程序：网络资源、存储资源、监视资源和更新资源。 互连用户体验与遥测组件使用 SSL 加密数据，并使用证书关联通过 HTTPS 将数据传输到 Microsoft 数据管理服务。

> [!IMPORTANT]
> 若要启用遥测数据流，必须在网络中开放端口 443 (HTTPS)。 互连用户体验与遥测组件连接到 Microsoft 数据管理服务（位于 `https://v10.events.data.microsoft.com`）。 互连用户体验与遥测组件还连接到 `https://settings-win.data.microsoft.com` 来下载配置信息。 其他诊断数据服务连接 `https://watson.telemetry.microsoft.com` 来报告错误。

## <a name="privacy-considerations"></a>隐私注意事项

ETW 服务将遥测数据发回到受保护的云存储。 最小特权原则指导对遥测数据的访问。 只有具有有效业务需求的 Azure 人员才能访问遥测数据。 除非客户自行要求，或者符合 [Azure 隐私声明](https://www.azure.cn/zh-cn/support/legal/privacy-statement/)中所述的受限目的，否则 Azure 不会与第三方共享客户个人数据。 与 OEM 和合作伙伴共享的业务报告包含聚合的匿名数据。 数据共享决策由 Azure 内部团队（包括隐私、法律和数据管理利益干系人）做出。

Azure 相信并实行信息最小化。 我们尽量只收集所需的信息，并且只在服务所需或进行分析时才存储这些信息。 许多有关 Azure Stack Hub 系统和 Azure 服务工作原则的信息在六个月内删除。 汇总或聚合的数据保留更长一段时间。

我们了解客户信息的隐私和安全都很重要。  Azure 采用深思熟虑的综合方法，在 Azure Stack Hub 中保护客户隐私和客户数据。 IT 管理员随时可以控制功能和隐私的自定义设置。 我们对于透明度和信任的承诺很明确：

- 我们向客户公开我们收集的数据类型。
- 企业客户有控制权 — 他们可以自定义自己的隐私设置。
- 我们将客户隐私和安全放在第一位。
- 我们以公开透明的方式使用遥测数据。
- 我们使用遥测数据来改进客户体验。

Azure 无意收集敏感数据，例如信用卡号、用户名和密码、电子邮件地址或类似的敏感信息。 如果我们确定敏感信息是无意中收集到的，我们会予以删除。

## <a name="examples-of-how-azure-uses-the-telemetry-data"></a>Azure 如何使用遥测数据的示例

遥测起着重要作用，可帮助我们快速找到并解决客户部署和配置的严重可靠性问题。 基于遥测数据的见解可帮助我们识别服务或硬件配置的问题。 Azure 从客户那里获取此数据以及推动生态系统改进的能力，可提高集成式 Azure Stack Hub 解决方案的质量。

遥测还能帮助 Azure 进一步了解客户如何部署组件、使用功能以及使用服务来实现业务目标。 这些见解有助于在直接影响客户体验和工作负荷的领域中指定工程投资的优先级。

示例包括：与 Azure Stack Hub 角色关联的容器、存储和网络配置的客户用法。 我们还使用见解来推动 Azure Stack Hub 管理和监视解决方案的改进与智能化。 这些改进可让客户更轻松地诊断问题，减少向 Azure 拨打支持电话的次数，从而节省资金。

## <a name="manage-telemetry-collection"></a>管理遥测数据的收集

我们不建议在组织中关闭遥测。 但是，在某些情况下有必要关闭遥测。

在这些情况下，可以在部署 Azure Stack Hub 之前使用注册表设置或者在部署 Azure Stack Hub 之后使用遥测终结点，来配置发送到 Azure 的遥测级别。

### <a name="telemetry-levels-and-data-collection"></a>遥测级别和数据收集

在更改遥测设置之前，应该了解遥测级别和每个级别收集哪些数据。

遥测设置分为四个累积级别 (0-3)，其分类如下：

**0（安全）**</br>
仅限安全数据。 确保操作系统安全所需的信息。 这包括有关互连用户体验和遥测组件设置以及 Windows Defender 的数据。 在此级别不会发出任何特定于 Azure Stack Hub 的遥测数据。

**1（基本）**</br>
安全数据，以及基本运行状况和质量数据。 基本设备信息，包括：质量相关的数据、应用兼容性、应用用量数据，以及来自“安全”级别的数据。  将遥测级别设置为“基本”可启用 Azure Stack Hub 遥测。 在此级别收集的数据包括：

- *基本设备信息*：提供生态系统中本机和虚拟 Windows Server 2016 实例的类型与配置相关的信息。 这包括：

  - 计算机属性，例如 OEM 和型号。
  - 网络属性，例如网络适配器的数目和速度。
  - 处理器和内存属性，例如核心数，以及安装的内存量。
  - 存储属性，例如驱动器的数目、类型和大小。

- *遥测功能*，包括已上传事件、已删除事件的百分比，以及数据上次上传时间。
- *质量相关信息*，帮助 Azure 基本了解 Azure Stack Hub 的运行情况。 例如，针对特定硬件配置发出的严重警报计数。
- *兼容性数据*，帮助了解系统和虚拟机 (VM) 上已安装哪些资源提供程序。 此数据用于识别潜在的兼容性问题。

**2（增强）**</br>
其他见解，包括：操作系统和 Azure Stack Hub 服务的用法、性能、高级可靠性数据，以及来自“安全”和“基本”级别的数据。  

> [!NOTE]
> 这是默认的遥测设置。

**3（完整）**</br>
识别及帮助解决问题所需的全部数据，加上来自“安全”、“基本”和“增强”级别的数据。   

> [!IMPORTANT]
> 这些遥测级别只适用于 Microsoft Azure Stack Hub 组件。 Azure Stack Hub 硬件合作伙伴在硬件生命周期主机中运行的非 Azure 软件组件和服务可能与这些遥测级别以外的云服务通信。 应该咨询 Azure Stack Hub 硬件解决方案提供商，以了解其遥测策略，以及如何启用或禁用。

关闭 Windows 和 Azure Stack Hub 遥测也会禁用 SQL 遥测。 有关 Windows Server 遥测设置的含义的详细信息，请参阅 [Windows 遥测白皮书](https://aka.ms/winservtelemetry)。

### <a name="asdk-set-the-telemetry-level-in-the-windows-registry"></a>ASDK：在 Windows 注册表中设置遥测级别

在部署 Azure Stack Hub 之前，可以使用 Windows 注册表编辑器在物理主机上手动设置遥测级别。 如果管理策略（例如组策略）已存在，它会覆盖此注册表设置。

在开发工具包主机上部署 Azure Stack Hub 之前，请先引导至 CloudBuilder.vhdx，然后在权限提升的 PowerShell 窗口中运行以下脚本：

```powershell
### Get current AllowTelemetry value on DVM Host
(Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" `
-Name AllowTelemetry).AllowTelemetry
### Set & Get updated AllowTelemetry value for ASDK-Host
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" `
-Name "AllowTelemetry" -Value '0' # Set this value to 0,1,2,or3.  
(Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" `
-Name AllowTelemetry).AllowTelemetry
```

### <a name="asdk-and-multi-node-enable-or-disable-telemetry-after-deployment"></a>ASDK 和多节点：在部署后启用或禁用遥测

若要在部署之后启用或禁用遥测，需要能够访问 ERCS VM 上公开的特权终结点 (PEP)。

- 若要启用：`Set-Telemetry -Enable`
- 若要禁用：`Set-Telemetry -Disable`

PARAMETER 详细信息：
- `.PARAMETER Enable` - 启用遥测数据上传
- `.PARAMETER Disable` - 禁用遥测数据上传  

**用于启用遥测的脚本：**

```powershell
$ip = "<IP ADDRESS OF THE PEP VM>" # You can also use the machine name instead of IP here.
$pwd= ConvertTo-SecureString "<CLOUD ADMIN PASSWORD>" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("<DOMAIN NAME>\CloudAdmin", $pwd)
$psSession = New-PSSession -ComputerName $ip -ConfigurationName PrivilegedEndpoint -Credential $cred
Invoke-Command -Session $psSession {Set-Telemetry -Enable}
if($psSession)
{
    Remove-PSSession $psSession
}
```

**用于禁用遥测的脚本：**

```powershell
$ip = "<IP ADDRESS OF THE PEP VM>" # You can also use the machine name instead of IP here.
$pwd= ConvertTo-SecureString "<CLOUD ADMIN PASSWORD>" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("<DOMAIN NAME>\CloudAdmin", $pwd)
$psSession = New-PSSession -ComputerName $ip -ConfigurationName PrivilegedEndpoint -Credential $cred
Invoke-Command -Session $psSession {Set-Telemetry -Disable}
if($psSession)
{
    Remove-PSSession $psSession
}
```

## <a name="next-steps"></a>后续步骤

[将 Azure Stack Hub 注册到 Azure](azure-stack-registration.md)