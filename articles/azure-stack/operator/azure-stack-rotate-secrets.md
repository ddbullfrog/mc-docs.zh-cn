---
title: 轮换机密
titleSuffix: Azure Stack Hub
description: 了解如何在 Azure Stack Hub 中轮换机密。
author: WenJason
ms.service: azure-stack
ms.topic: how-to
origin.date: 06/29/2020
ms.date: 10/12/2020
ms.reviewer: ppacent
ms.author: v-jay
ms.lastreviewed: 08/15/2020
ms.openlocfilehash: 5ad3f55a0b8d57297ec6f8a8ea399a4a00fe090b
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91437575"
---
# <a name="rotate-secrets-in-azure-stack-hub"></a>在 Azure Stack Hub 中轮换机密

这些说明仅适用于 Azure Stack Hub 集成系统 1803 和更高版本。请勿在低于 1803 的版本上尝试使用机密轮换

本文提供了用于机密轮换的指南和 PowerShell 脚本，以帮助维护与 Azure Stack Hub 基础结构资源和服务之间的安全通信。 

## <a name="overview"></a>概述

Azure Stack Hub 使用机密来维护与基础结构资源和服务之间的安全通信。 为维护 Azure Stack Hub 基础结构的完整性，操作员需要能够轮换机密，轮换频率应与其组织的安全要求一致。

### <a name="internal-vs-external-secrets"></a>内部与外部机密

从 1811 版开始，内部证书和外部证书的机密轮换是分开的。

- **内部机密**：由 Azure Stack Hub 基础结构使用的证书、密码、安全字符串和密钥，无需 Azure Stack Hub 操作员的介入。

- **外部机密**：对外服务的基础结构服务证书，由 Azure Stack Hub 操作员提供。 外部机密包括以下服务的证书：

    - 管理员门户
    - 公共门户
    - 管理员 Azure 资源管理器
    - 全局 Azure 资源管理器
    - 管理员 Key Vault
    - 密钥保管库
    - 管理扩展主机
    - ACS（包括 Blob、表和队列存储）
    - ADFS*
    - Graph*
    
    \*仅当环境的标识提供者是 Active Directory 联合身份验证服务 (AD FS) 时才适用。

> [!Important]
> 所有其他安全密钥和字符串都是由管理员手动更新的。 这包括用户和管理员帐户密码、[网络交换机密码和权限](azure-stack-customer-defined.md)，以及基板管理控制器 (BMC) 凭据（[将在本文后面部分介绍](#update-the-bmc-credential)）。 
>
>此外，本文未介绍增值资源提供程序的机密轮换。 若要轮换这些机密，请改为参阅以下文章：
>
> - [轮换 Azure Stack Hub 上应用服务的机密和证书](app-service-rotate-certificates.md)
> - [MySQL 资源提供程序 - 轮换机密](azure-stack-mysql-resource-provider-maintain.md#secrets-rotation)
> - [SQL 资源提供程序 - 轮换机密](azure-stack-sql-resource-provider-maintain.md#secrets-rotation)

### <a name="expiration-alerts"></a>机密过期的警报

机密过期后的 30 天内，管理员门户中会生成以下警报：

- 挂起的服务帐户密码过期
- 挂起的内部证书过期
- 挂起的外部证书过期

完成以下部分的机密轮换步骤会解决这些警报。

> [!Note]
> 在 1811 之前版本的 Azure Stack Hub 环境中，可能会看到内部证书挂起或机密过期的警报。 这些警报并不正确，应将其忽略，且不运行内部机密轮换。 不正确的内部机密过期警报是已知问题，已在 1811 中得到解决。 除非环境处于活动状态的时间已达两年，否则内部机密不会过期。

### <a name="external-certificates-from-a-new-certificate-authority"></a>新的证书颁发机构颁发的外部证书

在以下上下文中，Azure Stack Hub 支持使用新证书颁发机构 (CA) 颁发的外部证书进行机密轮换：

|已安装的证书 CA|要轮换到的 CA|支持|支持的 Azure Stack Hub 版本|
|-----|-----|-----|-----|
|从自签名|到企业|支持|1903 和更高版本|
|从自签名|到自签名|不支持||
|从自签名|到公共<sup>*</sup>|支持|1803 和更高版本|
|从企业|到企业|。 从 1803-1903：只要客户使用与在部署时使用的相同的企业 CA，就可以支持|1803 和更高版本|
|从企业|到自签名|不支持||
|从企业|到公共<sup>*</sup>|支持|1803 和更高版本|
|从公共<sup>*</sup>|到企业|支持|1903 和更高版本|
|从公共<sup>*</sup>|到自签名|不支持||
|从公共<sup>*</sup>|到公共<sup>*</sup>|支持|1803 和更高版本|

<sup>*</sup>指示公共证书颁发机构属于 Windows 受信任根计划。 可在以下文章中找到完整列表：[参与者列表 - Microsoft 受信任根计划](/security/trusted-root/participants-list)。

## <a name="prerequisites"></a>先决条件

对于内部和外部机密的轮换：

1. 强烈建议你首先将 Azure Stack Hub 实例更新到最新版本。

    >[!IMPORTANT]
    > 对于 1811 之前的版本：
    > - 如果已执行机密轮换，必须在再次执行机密轮换之前将版本更新为 1811 或更高版本。 必须通过[特权终结点](azure-stack-privileged-endpoint.md)执行机密轮换，且需要有 Azure Stack Hub 操作员凭据。 如果你不知道是否已在环境中运行过机密轮换，在执行机密轮换之前，请先更新到 1811。
    > - 无需轮换机密即可添加扩展主机证书。 应该按照[准备 Azure Stack Hub 的扩展主机](azure-stack-extension-host-prepare.md)一文中的说明添加扩展主机证书。

2. 向用户通知计划内维护操作。 将普通的维护时间段尽量安排在非营业时间。 维护操作可能会同时影响用户工作负荷和门户操作。

3. 在机密轮换期间，操作员可能会注意到警报在打开后又自动关闭。 此行为是预期行为，可以忽略警报。 操作员可以使用 [Test-AzureStack PowerShell cmdlet](azure-stack-diagnostic-test.md) 来验证这些警报的有效性。 对于使用 System Center Operations Manager 监视 Azure Stack Hub 系统的操作人员来说，将系统置于维护模式将阻止这些警报到达其 ITSM 系统，但如果 Azure Stack Hub 系统无法访问，则将继续发出警报。

若要轮换外部机密，请完成以下附加先决条件：

1. 在轮换机密之前，请运行 **[Test-AzureStack](azure-stack-diagnostic-test.md)** 并确认所有测试输出都正常。
2. 准备新的替换性的外部证书集：
    - 新集必须与 [Azure Stack Hub PKI 证书要求](azure-stack-pki-certs.md)中所述的证书规范匹配。 
    - 可以使用[生成证书签名请求](azure-stack-get-pki-certs.md)中概述的步骤，生成需提交到证书颁发机构 (CA) 的证书签名请求 (CSR)，然后使用[准备 PKI 证书](azure-stack-prepare-pki-certs.md)中的步骤来准备这些证书，以便在 Azure Stack Hub 环境中使用。 
    - 请务必使用[验证 PKI 证书](azure-stack-validate-pki-certs.md)中概述的步骤来验证准备的证书
    - 请确保密码中没有特殊字符，例如 `*` 或 `)`。
    - 请确保 PFX 加密为 **TripleDES-SHA1**。 如果遇到问题，请参阅[修复 Azure Stack Hub PKI 证书的常见问题](azure-stack-remediate-certs.md#pfx-encryption)。
3. 将用于轮换的证书备份存储在安全的备份位置。 如果运行轮换时发生失败，请使用备份副本替换文件共享中的证书，然后重新运行轮换。 将备份副本保存在安全的备份位置。
4. 创建可从 ERCS VM 访问的文件共享。 该文件共享必须可供 **CloudAdmin** 标识读取和写入。
5. 在可以访问该文件共享的计算机上打开 PowerShell ISE 控制台。 导航到你的文件共享，你将在其中创建目录来放置外部证书。
6. 将 **[CertDirectoryMaker.ps1](https://www.aka.ms/azssecretrotationhelper)** 下载到在轮换期间可以访问的网络文件共享，并运行该脚本。 该脚本将创建一个文件夹结构，该结构符合 .\Certificates\AAD 或 .\Certificates\ADFS 的要求，具体取决于你的标识提供者。 你的文件夹结构必须以 **\\Certificates** 文件夹开始，后面仅跟有一个 **\\AAD** 或 **\\ADFS** 文件夹。 所有其他子目录都包含在前面的结构中。 例如：
    - 文件共享 = **\\\\\<IPAddress>\\\<ShareName>**
    - Azure AD 提供程序的证书根文件夹 = **\\Certificates\AAD**
    - 完整路径 = **\\\\\<IPAddress>\\\<ShareName>\Certificates\AAD**

    > [!IMPORTANT]
    > 稍后运行 `Start-SecretRotation` 时，它会验证文件夹结构。 不符合标准的文件夹结构会引发以下错误：
    >
    > ```powershell
    > Cannot bind argument to parameter 'Path' because it is null.
    > + CategoryInfo          : InvalidData: (:) [Test-Certificate], ParameterBindingValidationException
    > + FullyQualifiedErrorId : ParameterArgumentValidationErrorNullNotAllowed,Test-Certificate
    > + PSComputerName        : xxx.xxx.xxx.xxx
    > ```

7. 将在步骤 2 中创建的一组新的替换性的外部证书复制到在步骤 6 中创建的 **\Certificates\\\<IdentityProvider>** 目录。 对于 \<CertName\>，请务必遵循 `cert.<regionName>.<externalFQDN>` 格式。 

    下面是 Azure AD 标识提供者的文件夹结构示例：
    ```powershell
        <ShareName>
            │
            └───Certificates
                  └───AAD
                      ├───ACSBlob
                      │       <CertName>.pfx
                      │
                      ├───ACSQueue
                      │       <CertName>.pfx
                      │
                      ├───ACSTable
                      │       <CertName>.pfx
                      │
                      ├───Admin Extension Host
                      │       <CertName>.pfx
                      │
                      ├───Admin Portal
                      │       <CertName>.pfx
                      │
                      ├───ARM Admin
                      │       <CertName>.pfx
                      │
                      ├───ARM Public
                      │       <CertName>.pfx
                      │
                      ├───KeyVault
                      │       <CertName>.pfx
                      │
                      ├───KeyVaultInternal
                      │       <CertName>.pfx
                      │
                      ├───Public Extension Host
                      │       <CertName>.pfx
                      │
                      └───Public Portal
                              <CertName>.pfx

    ```

## <a name="rotate-external-secrets"></a>轮换外部机密

完成以下步骤来轮换外部机密：

1. 使用以下 PowerShell 脚本来轮换机密。 此脚本要求访问特权终结点 (PEP) 会话。 可在托管 PEP 的虚拟机 (VM) 上通过远程 PowerShell 会话来访问 PEP。 如果使用集成系统，则有三个 PEP 实例，每个实例在不同主机上的 VM（Prefix-ERCS01、Prefix-ERCS02 或 Prefix-ERCS03）中运行。 如果你使用的是 ASDK，则此 VM 名为 AzS-ERCS01。 在运行之前更新 `<placeholder>` 值：

    ```powershell
    # Create a PEP Session
    winrm s winrm/config/client '@{TrustedHosts= "<IP_address_of_ERCS>"}'
    $PEPCreds = Get-Credential
    $PEPSession = New-PSSession -ComputerName <IP_address_of_ERCS_Machine> -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

    # Run Secret Rotation
    $CertPassword = ConvertTo-SecureString "<Cert_Password>" -AsPlainText -Force
    $CertShareCreds = Get-Credential
    $CertSharePath = "<Network_Path_Of_CertShare>"
    Invoke-Command -Session $PEPSession -ScriptBlock {
        Start-SecretRotation -PfxFilesPath $using:CertSharePath -PathAccessCredential $using:CertShareCreds -CertificatePassword $using:CertPassword
    }
    Remove-PSSession -Session $PEPSession
    ```

    此脚本执行以下步骤：

    - 使用 **CloudAdmin** 帐户创建具有[特权终结点](azure-stack-privileged-endpoint.md)的 PowerShell 会话，并将会话存储为变量。 此变量在下一步用作参数。 

    - 运行 [Invoke-Command](https://docs.microsoft.com/powershell/module/microsoft.powershell.core/Invoke-Command?view=powershell-5.1)，将 PEP 会话变量作为 `-Session` 参数传递。

    - 使用以下参数在 PEP 会话中运行 `Start-SecretRotation`：
        - `-PfxFilesPath`：之前创建的 Certificates 目录的网络路径。  
        - `-PathAccessCredential`：针对共享的凭据的 PSCredential 对象。
        - `-CertificatePassword`：创建的所有 pfx 证书文件使用的密码安全字符串。

2. 外部机密轮换需要大约一小时。 成功完成后，控制台会显示 `ActionPlanInstanceID ... CurrentStatus: Completed`，后跟 `DONE`。 从共享（在先决条件部分创建）中删除证书，将其存储在安全的备份位置。

    > [!Note]
    > 如果机密轮换失败，请按照错误消息中的说明操作，结合 `-ReRun` 参数重新运行 `Start-SecretRotation`。
    >
    >```powershell
    >Start-SecretRotation -ReRun
    >```  
    >
    >如果遇到反复的机密轮换失败，请联系技术支持。

## <a name="rotate-internal-secrets"></a>轮换内部机密

仅当你怀疑某个机密已泄露或收到机密过期的警报时，才需要轮换内部机密。 在 1811 之前的版本中，你可能会看到内部证书挂起或机密过期的警报。 这些警报不准确，应该将其忽略，这是 1811 中已解决的已知问题。 除非环境处于活动状态的时间已达两年，否则内部机密不会过期。

请参考[轮换外部机密](#rotate-external-secrets)的步骤 2 中的 PowerShell 脚本。 此脚本提供了一个示例。你可以通过进行一些更改来运行以下步骤，以便改编该示例，使其适合内部机密轮换：

1. 在“运行机密轮换”部分，将 `-Internal` 参数添加到 [Start-SecretRotation cmdlet](/azure-stack/reference/pep-2002/start-secretrotation)，例如：

    ```powershell
    # Run Secret Rotation
    ...
    Invoke-Command -Session $PEPSession -ScriptBlock {
        Start-SecretRotation -Internal
    }
    ...
    ```

    > [!Note]
    > 1811 之前的版本不需要 `-Internal` 标志。 

3. 成功完成后，控制台会显示 `ActionPlanInstanceID ... CurrentStatus: Completed`，后跟 `DONE`

    > [!Note]
    > 如果机密轮换失败，请按照错误消息中的说明操作，并使用 `-Internal` 和 `-ReRun` 参数重新运行 `Start-SecretRotation`。  
    >
    >```powershell
    >Start-SecretRotation -Internal -ReRun
    >```
    >
    > 如果遇到反复的机密轮换失败，请联系技术支持。

## <a name="update-the-bmc-credential"></a>更新 BMC 凭据

基板管理控制器监视服务器的物理状态。 请咨询原始设备制造商 (OEM) 硬件供应商，以获取有关更新 BMC 的用户帐户名和密码的说明。

>[!NOTE]
> OEM 可能提供附加的管理应用。 更新其他管理应用的用户名或密码不会影响 BMC 用户名或密码。 

1. 按照 OEM 说明在 Azure Stack Hub 的物理服务器上更新 BMC。 环境中每个 BMC 的用户名和密码必须相同。 BMC 用户名不能超过 16 个字符。

1. 不再需要按照 OEM 的说明先在 Azure Stack Hub 物理服务器上更新 BMC 凭据。 环境中每个 BMC 的用户名和密码必须相同，并且不能超过 16 个字符。 

2. 在 Azure Stack Hub 会话中打开特权终结点。 有关说明，请参阅[使用 Azure Stack Hub 中的特权终结点](azure-stack-privileged-endpoint.md)。 

3. 在 PowerShell 提示符更改为 `[IP address or ERCS VM name]: PS>` 或 `[azs-ercs01]: PS>` 后，根据环境通过运行 `Invoke-Command` 来运行 `Set-BmcCredential`。 如果你将可选的 `-BypassBMCUpdate` 参数用于 `Set-BMCCredential`，则 BMC 中的凭据不会更新。 只有 Azure Stack Hub 内部数据存储会更新。请将特权终结点会话变量作为参数进行传递。 

    下面是一个示例 PowerShell 脚本，它会提示你输入用户名和密码： 

    ```powershell
    # Interactive Version
    $PEPIp = "<Privileged Endpoint IP or Name>" # You can also use the machine name instead of IP here.
    $PEPCreds = Get-Credential "<Domain>\CloudAdmin" -Message "PEP Credentials"
    $NewBmcPwd = Read-Host -Prompt "Enter New BMC password" -AsSecureString
    $NewBmcUser = Read-Host -Prompt "Enter New BMC user name"

    $PEPSession = New-PSSession -ComputerName $PEPIp -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

    Invoke-Command -Session $PEPSession -ScriptBlock {
        # Parameter BmcPassword is mandatory, while the BmcUser parameter is optional.
        Set-BmcCredential -BmcPassword $using:NewBmcPwd -BmcUser $using:NewBmcUser
    }
    Remove-PSSession -Session $PEPSession
    ```

    你还可以在变量中对用户名和密码进行编码，这可能会降低安全性：

    ```powershell
    # Static Version
    $PEPIp = "<Privileged Endpoint IP or Name>" # You can also use the machine name instead of IP here.
    $PEPUser = "<Privileged Endpoint user for example Domain\CloudAdmin>"
    $PEPPwd = ConvertTo-SecureString "<Privileged Endpoint Password>" -AsPlainText -Force
    $PEPCreds = New-Object System.Management.Automation.PSCredential ($PEPUser, $PEPPwd)
    $NewBmcPwd = ConvertTo-SecureString "<New BMC Password>" -AsPlainText -Force
    $NewBmcUser = "<New BMC User name>"

    $PEPSession = New-PSSession -ComputerName $PEPIp -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

    Invoke-Command -Session $PEPSession -ScriptBlock {
        # Parameter BmcPassword is mandatory, while the BmcUser parameter is optional.
        Set-BmcCredential -BmcPassword $using:NewBmcPwd -BmcUser $using:NewBmcUser
    }
    Remove-PSSession -Session $PEPSession
    ```

## <a name="reference-start-secretrotation-cmdlet"></a>参考：Start-SecretRotation cmdlet

[Start-SecretRotation cmdlet](/azure-stack/reference/pep-2002/start-secretrotation) 轮换 Azure Stack Hub 系统的基础结构机密。 仅可通过使用 `Invoke-Command` 脚本块在 `-Session` 参数中传递 PEP 会话，对 Azure Stack Hub 特权终结点执行此 cmdlet。 默认情况下，它只轮换所有外部网络基础结构终结点的证书。

| 参数 | 类型 | 必须 | 位置 | 默认 | 说明 |
|--|--|--|--|--|--|
| `PfxFilesPath` | String  | False  | 名为  | 无  | 包含所有外部网络终结点证书的 **\Certificates** 目录的文件共享路径。 仅当轮换外部机密时才需要。 结尾目录必须是 **\Certificates**。 |
| `CertificatePassword` | SecureString | False  | 名为  | 无  | -PfXFilesPath 中提供的所有证书的密码。 如果在轮换外部机密时提供了 PfxFilesPath，则是必需的值。 |
| `Internal` | String | False | 名为 | 无 | 每当 Azure Stack Hub 操作员想要轮换内部基础结构机密时，都必须使用 Internal 标志。 |
| `PathAccessCredential` | PSCredential | False  | 名为  | 无  | 包含所有外部网络终结点证书的 **\Certificates** 目录的文件共享的 PowerShell 凭据。 仅当轮换外部机密时才需要。  |
| `ReRun` | SwitchParameter | False  | 名为  | 无  | 每当机密轮换尝试失败后重新进行尝试时，都必须使用此项。 |

### <a name="syntax"></a>语法

#### <a name="for-external-secret-rotation"></a>对于外部机密轮换

```powershell
Start-SecretRotation [-PfxFilesPath <string>] [-PathAccessCredential <PSCredential>] [-CertificatePassword <SecureString>]  
```

#### <a name="for-internal-secret-rotation"></a>对于内部机密轮换

```powershell
Start-SecretRotation [-Internal]  
```

#### <a name="for-external-secret-rotation-rerun"></a>对于外部机密轮换 rerun

```powershell
Start-SecretRotation [-ReRun]
```

#### <a name="for-internal-secret-rotation-rerun"></a>对于内部机密轮换 rerun

```powershell
Start-SecretRotation [-ReRun] [-Internal]
```

### <a name="examples"></a>示例

#### <a name="rotate-only-internal-infrastructure-secrets"></a>仅轮换内部基础结构机密

此命令必须通过 Azure Stack Hub [环境的特权终结点](azure-stack-privileged-endpoint.md)运行。

```powershell
PS C:\> Start-SecretRotation -Internal
```

此命令轮换向 Azure Stack Hub 内部网络公开的所有基础结构机密。

#### <a name="rotate-only-external-infrastructure-secrets"></a>仅轮换外部基础结构机密  

```powershell
# Create a PEP Session
winrm s winrm/config/client '@{TrustedHosts= "<IP_address_of_ERCS>"}'
$PEPCreds = Get-Credential
$PEPSession = New-PSSession -ComputerName <IP_address_of_ERCS> -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

# Create Credentials for the fileshare
$CertPassword = ConvertTo-SecureString "<CertPasswordHere>" -AsPlainText -Force
$CertShareCreds = Get-Credential
$CertSharePath = "<NetworkPathOfCertShare>"
# Run Secret Rotation
Invoke-Command -Session $PEPSession -ScriptBlock {  
    Start-SecretRotation -PfxFilesPath $using:CertSharePath -PathAccessCredential $using:CertShareCreds -CertificatePassword $using:CertPassword
}
Remove-PSSession -Session $PEPSession
```

此命令轮换用于 Azure Stack Hub 外部网络基础结构终结点的 TLS 证书。

#### <a name="rotate-internal-and-external-infrastructure-secrets-pre-1811-only"></a>轮换内部和外部基础结构机密（仅限 **1811 以前**的版本）

> [!IMPORTANT]
> 此命令仅适用于 Azure Stack Hub **1811 以前**的版本，因为轮换将会针对内部和外部证书分开进行。
>
> **在 1811 以上的版本中，不再能够同时轮换内部和外部证书！** 

```powershell
# Create a PEP Session
winrm s winrm/config/client '@{TrustedHosts= "<IP_address_of_ERCS>"}'
$PEPCreds = Get-Credential
$PEPSession = New-PSSession -ComputerName <IP_address_of_ERCS> -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

# Create Credentials for the fileshare
$CertPassword = ConvertTo-SecureString "<CertPasswordHere>" -AsPlainText -Force
$CertShareCreds = Get-Credential
$CertSharePath = "<NetworkPathOfCertShare>"
# Run Secret Rotation
Invoke-Command -Session $PEPSession -ScriptBlock {
    Start-SecretRotation -PfxFilesPath $using:CertSharePath -PathAccessCredential $using:CertShareCreds -CertificatePassword $using:CertPassword
}
Remove-PSSession -Session $PEPSession
```

此命令轮换向 Azure Stack Hub 内部网络公开的基础结构机密，以及用于 Azure Stack Hub 的外部网络基础结构终结点的 TLS 证书。 Start-SecretRotation 轮换堆栈生成的所有机密，由于提供了证书，因此也会轮换外部终结点证书。  

## <a name="next-steps"></a>后续步骤

[详细了解 Azure Stack Hub 安全性](azure-stack-security-foundations.md)
