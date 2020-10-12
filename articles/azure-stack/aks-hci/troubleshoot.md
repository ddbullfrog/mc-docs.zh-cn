---
title: 故障排除
description: Azure Stack HCI 上的 Azure Kubernetes 服务故障排除指南
author: WenJason
ms.topic: how-to
ms.service: azure-stack
origin.date: 09/22/2020
ms.date: 10/12/2020
ms.author: v-jay
ms.openlocfilehash: b78fd2b6fe46b4ee8a2e878820c188ebc90ff18a
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91451214"
---
# <a name="troubleshooting-azure-kubernetes-service-on-azure-stack-hci"></a>Azure Stack HCI 上的 Azure Kubernetes 服务故障排除

使用 Azure Stack HCI 上的 Azure Kubernetes 服务创建或管理 Kubernetes 群集时，可能偶尔会遇到问题。 下面是可帮助你解决这些问题的故障排除指南。 

## <a name="troubleshooting-azure-stack-hci"></a>Azure Stack HCI 故障排除
若要对 Azure Stack HCI 群集中所有服务器上的网络和存储 QoS（服务质量）设置的群集验证报表进行故障排除，并验证是否已定义重要规则，请访问[对群集验证报表进行故障排除](/azure-stack/hci/manage/validate-qos)。

若要排查 CredSSP 的问题，请访问 [CredSSP 故障排除](/azure-stack/hci/manage/troubleshoot-credssp)。

## <a name="troubleshooting-windows-admin-center"></a>Windows Admin Center 疑难解答
此产品目前为公共预览状态，这意味着它仍处于开发阶段。 目前，Windows Admin Center Azure Kubernetes 服务扩展存在几个问题： 
* 目前，用来设置 Azure Stack HCI 上的 Azure Kubernetes 服务的系统群集中的每个服务器都必须是受信任的服务器。 这意味着，Windows Admin Center 必须能够在群集中的每个（而不只是一个或数个）服务器上执行 CredSSP 操作。 
* 如果遇到“`msft.sme.aks couldn't load`”错误，并且该错误指出加载区块失败，请使用最新版 Edge 或 Google Chrome 并重试。
* 在启动 Azure Kubernetes 服务主机设置向导或“创建 Kubernetes 群集”向导之前，应通过 Windows Admin Center 登录到 Azure。 可能需要在工作流中重新签名。 如果在通过 Windows Admin Center 登录到 Azure 时遇到困难，请尝试从其他源（如 [Azure 门户](https://portal.azure.cn/)）登录到 Azure 帐户。 如果仍然遇到问题，请在与支持部门联系之前，查看 [Windows Admin Center 已知问题](https://docs.microsoft.com/windows-server/manage/windows-admin-center/support/known-issues)一文。
* 在通过 Windows Admin Center 进行的 Azure Stack HCI 上 Azure Kubernetes 服务部署的当前迭代中，只有设置 Azure Kubernetes 服务主机的用户才可以在系统上创建 Kubernetes 群集。 若要解决此问题，请将 `.wssd` 文件夹从设置 Azure Kubernetes 服务主机的用户配置文件复制到将要启动新 Kubernetes 群集的用户配置文件。
* 如果在任一向导中收到一个有关错误配置的错误，请执行群集清理操作。 这可能涉及到删除 `C:\Program Files\AksHci\mocctl.exe` 文件。
* 若要使 CredSSP 在群集创建向导中成功地发挥作用，Windows Admin Center 必须由同一帐户安装和使用。 如果使用一个帐户进行安装，然后尝试通过另一个帐户来使用它，则会导致错误。
* 在群集部署过程中，helm.zip 文件传输可能会出现问题。 这通常会导致一个错误，其中指出 helm.zip 文件的路径不存在或无效。 若要解决此问题，请返回并重试部署。
* 如果部署在很长时间内处于挂起状态，则表明可能存在 CredSSP 问题或连接性问题。 请尝试通过执行以下步骤来排查部署问题： 
    1.  在运行 WAC 的计算机上的 PowerShell 窗口中运行以下命令： 
    ```PowerShell
    Enter-PSSession <servername>
    ```
    2.  如果此命令成功，则表明你可以连接到服务器，没有连接性问题。
    
    3.  如果遇到 CredSSP 问题，请运行以下命令来测试网关计算机与目标计算机之间的信任： 
    ```PowerShell
    Enter-PSSession -ComputerName <server> -Credential company\administrator -Authentication CredSSP
    ``` 
    还可以运行以下命令来测试在访问本地网关时的信任： 
    ```PowerShell
    Enter-PSSession -computer localhost -credential (Get-Credential)
    ``` 
* 如果使用的是 Azure Arc 且有多个租户 ID，请在部署之前运行以下命令指定所需租户。 否则，可能会导致部署失败。

```Azure CLI
az login -tenant <tenant>
```
* 如果刚刚创建了新的 Azure 帐户，且尚未在网关计算机上登录到该帐户，则可能会在将 WAC 网关注册到 Azure 时遇到问题。 若要缓解此问题，请在另一个浏览器标签页或窗口中登录到 Azure 帐户，然后将 WAC 网关注册到 Azure。

### <a name="creating-windows-admin-center-logs"></a>创建 Windows Admin Center 日志
在 Windows Admin Center 中报告问题时，可以附加日志，以便开发团队诊断问题。 Windows Admin Center 中的错误通常分为两种形式：在运行 Windows Admin Center 的计算机上的事件查看器中显示的事件，或在浏览器控制台中显示的 JavaScript 问题。 若要收集 Windows Admin Center 的日志，请使用公共预览版包中提供的 `Get-SMEUILogs.ps1` 脚本。 
 
若要使用该脚本，请在存储脚本的文件目录中运行以下命令： 
 
```PowerShell
./Get-SMEUILogs.ps1 -ComputerNames [comp1, comp2, etc.] -Destination [comp3] -HoursAgo [48] -NoCredentialPrompt
```
 
该命令有以下参数：
 
* **-ComputerNames**：要从中收集日志的计算机的列表。
* **-Destination**：要将日志聚合到的计算机。
* **-HoursAgo**：此参数定义你想要自脚本运行起的多少小时之前收集日志。
* **-NoCredentialPrompt**：这是一个开关，用于在当前环境中关闭凭据提示并使用默认凭据。
 
如果在运行此脚本时遇到困难，可以运行以下命令来查看帮助文本： 
 
```PowerShell
GetHelp .\Get-SMEUILogs.ps1 -Examples
```

## <a name="troubleshooting-windows-worker-nodes"></a>Windows 工作器节点故障排除 
若要登录到 Windows 工作器节点，请先通过运行 `kubectl get` 获取节点的 IP 地址，并记下 `EXTERNAL-IP` 值：

```PowerShell
kubectl get nodes -o wide
``` 
使用 `ssh Administrator@ip` 通过 SSH 登录到节点。 通过 ssh 登录到节点后，可以运行 `net user administrator *` 来更新管理员的密码。 

## <a name="troubleshooting-linux-worker-nodes"></a>Linux 工作器节点故障排除 
若要登录到 Linux 工作器节点，请先通过运行 `kubectl get` 获取节点的 IP 地址，并记下 `EXTERNAL-IP` 值：

```PowerShell
kubectl get nodes -o wide
``` 
使用 `ssh clouduser@ip` 通过 SSH 登录到节点。 


## <a name="next-steps"></a>后续步骤
如果仍然在使用 Azure Stack HCI 上的 Azure Kubernetes 服务时遇到问题，请通过 [GitHub](https://aka.ms/aks-hci-issues) 提交 bug。  
