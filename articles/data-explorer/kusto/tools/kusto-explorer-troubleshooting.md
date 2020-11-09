---
title: 排查 Kusto.Explorer 中的常见问题
description: 了解有关安装和运行 Kusto.Explorer 的常见问题及其解决方案
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: conceptual
origin.date: 04/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: ab3f8ceb81263594452a00e8cd73e95bbee3d6dc
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104517"
---
# <a name="troubleshooting"></a>故障排除

本文档介绍运行和使用 Kusto.Explorer 时遇到的常见问题，并提供解决方案。 本文档还介绍了[如何重置 Kusto.Explorer](#reset-kustoexplorer)。

## <a name="kustoexplorer-fails-to-start"></a>Kusto.Explorer 无法启动

### <a name="kustoexplorer-shows-error-dialog-during-or-after-start-up"></a>Kusto.Explorer 在启动过程中或启动后显示错误对话

#### <a name="symptom"></a>症状

在启动时，Kusto.Explorer 显示 `InvalidOperationException` 错误。

#### <a name="possible-solution"></a>可能的解决方法

此错误可能表明操作系统已损坏或缺少某些必需模块。
若要检查丢失或已损坏的系统文件，请按照下面所述的步骤进行操作：   
[https://support.microsoft.com/help/929833/use-the-system-file-checker-tool-to-repair-missing-or-corrupted-system](https://support.microsoft.com/help/929833/use-the-system-file-checker-tool-to-repair-missing-or-corrupted-system)

## <a name="kustoexplorer-always-downloads-even-when-there-are-no-updates"></a>即使没有更新，Kusto.Explorer 也总要下载

#### <a name="symptom"></a>症状

每次打开 Kusto.Explorer 时，系统都会提示安装新版本。 Kusto.Explorer 会下载整个包，而不更新已安装的版本。

#### <a name="possible-solution"></a>可能的解决方法

此症状可能是本地 ClickOnce 存储中的损坏造成的。 你可以通过在提升的命令提示符中运行以下命令来清除本地 ClickOnce 存储区。

> [!IMPORTANT]
> 1. 如果存在 ClickOnce 应用程序或 `dfsvc.exe` 的任何其他实例，请在运行此命令前终止它们。
> 1. 所有 ClickOnce 应用将在下次运行时自动重新安装（只要你有权访问存储在应用程序快捷方式中的原始安装位置）。 不会删除应用快捷方式。

```kusto
rd /q /s %userprofile%\appdata\local\apps\2.0
```

尝试再次从其中一个[安装镜像](kusto-explorer.md#installing-kustoexplorer)安装 Kusto.Explorer。

### <a name="clickonce-error-cannot-start-application"></a>ClickOnce 错误：无法启动应用程序

#### <a name="symptoms"></a>症状

程序无法启动，并显示以下错误之一： 
* `External component has thrown an exception`
* `Value does not fall within the expected range`
* `The application binding data format is invalid.` 
* `Exception from HRESULT: 0x800736B2`
* `The referenced assembly is not installed on your system. (Exception from HRESULT: 0x800736B3)`

可通过单击以下错误对话框中的 `Details` 来浏览错误详细信息：

![ClickOnce 错误](./Images/kusto-explorer-troubleshooting/clickonce-err-1.png)

```kusto
Following errors were detected during this operation.
    * System.ArgumentException
        - Value does not fall within the expected range.
        - Source: System.Deployment
        - Stack trace:
            at System.Deployment.Application.NativeMethods.CorLaunchApplication(UInt32 hostType, String applicationFullName, Int32 manifestPathsCount, String[] manifestPaths, Int32 activationDataCount, String[] activationData, PROCESS_INFORMATION processInformation)
            at System.Deployment.Application.ComponentStore.ActivateApplication(DefinitionAppId appId, String activationParameter, Boolean useActivationParameter)
            at System.Deployment.Application.SubscriptionStore.ActivateApplication(DefinitionAppId appId, String activationParameter, Boolean useActivationParameter)
            at System.Deployment.Application.ApplicationActivator.Activate(DefinitionAppId appId, AssemblyManifest appManifest, String activationParameter, Boolean useActivationParameter)
            at System.Deployment.Application.ApplicationActivator.ProcessOrFollowShortcut(String shortcutFile, String& errorPageUrl, TempFile& deployFile)
            at System.Deployment.Application.ApplicationActivator.PerformDeploymentActivation(Uri activationUri, Boolean isShortcut, String textualSubId, String deploymentProviderUrlFromExtension, BrowserSettings browserSettings, String& errorPageUrl)
            at System.Deployment.Application.ApplicationActivator.ActivateDeploymentWorker(Object state)
```

#### <a name="proposed-solution-steps"></a>建议的解决方案步骤

1. 使用 `Programs and Features` (`appwiz.cpl`) 卸载 Kusto.Explorer。

1. 尝试运行 `CleanOnlineAppCache`，然后尝试再次安装 Kusto.Explorer。 
    从提升的命令提示符执行以下操作： 
    
    ```kusto
    rundll32 %windir%\system32\dfshim.dll CleanOnlineAppCache
    ```

    再次从其中一个[安装镜像](kusto-explorer.md#installing-kustoexplorer)安装 Kusto.Explorer。

1. 如果应用程序仍未启动，请删除本地 ClickOnce 存储。 所有 ClickOnce 应用将在下次运行时自动重新安装（只要你有权访问存储在应用程序快捷方式中的原始安装位置）。 不会删除应用快捷方式。

    从提升的命令提示符执行以下操作：

    ```kusto
    rd /q /s %userprofile%\appdata\local\apps\2.0
    ```

    再次从其中一个[安装镜像](kusto-explorer.md#installing-kustoexplorer)安装 Kusto.Explorer

1. 如果应用程序仍未启动：
    1. 删除临时部署文件。
    1. 重命名 Kusto.Explorer 本地 AppData 文件夹。

        从提升的命令提示符执行以下操作：

        ```kusto
        rd /s/q %userprofile%\AppData\Local\Temp\Deployment
        ren %LOCALAPPDATA%\Kusto.Explorer Kusto.Explorer.bak
        ```

    1. 再次从其中一个[安装镜像](kusto-explorer.md#installing-kustoexplorer)安装 Kusto.Explorer

    1. 若要从 Kusto.Explorer.bak 还原连接，请从提升的命令提示符执行以下操作：

        ```kusto
        copy %LOCALAPPDATA%\Kusto.Explorer.bak\User*.xml %LOCALAPPDATA%\Kusto.Explorer
        ```

#### <a name="enabling-clickonce-verbose-logging"></a>启用 ClickOnce 详细日志记录

1. 如果应用程序仍未启动：
    1. 在以下位置创建 LogVerbosityLevel 字符串值 1，以[启用详细 ClickOnce 日志记录](https://docs.microsoft.com/visualstudio/deployment/how-to-specify-verbose-log-files-for-clickonce-deployments)：

        ```kusto
        HKEY_CURRENT_USER\Software\Classes\Software\Microsoft\Windows\CurrentVersion\Deployment
        ```
    
    1. 再次重现它。
    1. 将详细输出发送到 KEBugReport@microsoft.com。 

### <a name="clickonce-error-your-administrator-has-blocked-this-application-because-it-potentially-poses-a-security-risk-to-your-computer"></a>ClickOnce 错误：管理员已阻止此应用程序，因为它可能会给你的计算机带来安全风险

#### <a name="symptom"></a>症状 
应用程序安装失败，并出现以下错误之一：
* `Your administrator has blocked this application because it potentially poses a security risk to your computer`.
* `Your security settings do not allow this application to be installed on your computer.`

#### <a name="solution"></a>解决方案

此症状可能是因为另一个应用程序正在替代默认的 ClickOnce 信任提示行为。 
1. 查看默认配置设置。
1. 将你的配置设置与计算机上的实际设置进行比较。
1. 必要时请重置配置设置，如[此操作说明文章中](https://docs.microsoft.com/visualstudio/deployment/how-to-configure-the-clickonce-trust-prompt-behavior)所述。

### <a name="cleanup-application-data"></a>清除应用程序数据

有时，如果先前的故障排除步骤无法启动 Kusto.Explorer，清理本地存储的数据可能会有所帮助。

可在此处找到 Kusto.Explorer 应用程序存储的数据：`C:\Users\[your username]\AppData\Local\Kusto.Explorer`。

> [!NOTE]
> 清除数据会导致丢失已打开的选项卡（“恢复”文件夹）、已保存的连接（“连接”文件夹）和应用程序设置（“UserSettings”文件夹）。

## <a name="reset-kustoexplorer"></a>重置 Kusto.Explorer

如果需要，可以完全重置 Kusto.Explorer。 以下过程介绍如何逐步重置 Kusto.Explorer，直到将其从计算机中删除且必须从头开始安装。

1. 在 Windows 中，打开“更改或删除程序”（也称为“程序和功能”） 。
1. 选择以 `Kusto.Explorer` 开头的每个项。
1. 选择“卸载” 。

   如果此过程未能卸载应用程序（ClickOnce 应用程序的已知问题），请参阅[本文以获取说明](https://stackoverflow.com/questions/10896223/how-do-i-completely-uninstall-a-clickonce-application-from-my-computer)。

1. 删除文件夹 `%LOCALAPPDATA%\Kusto.Explorer`，该文件夹用于删除所有连接、历史记录等。

1. 删除文件夹 `%APPDATA%\Kusto`，该文件夹用于删除 Kusto.Explorer 令牌缓存。 需要对所有群集重新进行身份验证。

也可以还原到特定版本的 Kusto.Explorer：

1. 运行 `appwiz.cpl`。
1. 选择“Kusto.Explorer”，然后选择“卸载/更改” 。
1. 选择“将应用程序还原到其先前的状态”。

## <a name="next-steps"></a>后续步骤

* 了解 [Kusto.Explorer 用户界面](kusto-explorer.md#overview-of-the-user-interface)
* 了解如何[从命令行运行 Kusto.Explorer](kusto-explorer-using.md#kustoexplorer-command-line-arguments)
* 了解 [Kusto 查询语言 (KQL)](../query/index.md)