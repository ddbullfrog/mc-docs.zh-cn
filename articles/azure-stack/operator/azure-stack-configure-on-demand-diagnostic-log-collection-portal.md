---
title: 立即发送 Azure Stack Hub 诊断日志
description: 了解如何使用管理员门户或 PowerShell 脚本在 Azure Stack Hub 中按需收集诊断日志。
author: WenJason
ms.service: azure-stack
ms.topic: article
origin.date: 08/24/2020
ms.date: 10/12/2020
ms.author: v-jay
ms.reviewer: shisab
ms.lastreviewed: 08/24/2020
ms.openlocfilehash: e7fd4bdd940407cadf2ce3f337ccc4b171d118b1
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91437733"
---
# <a name="send-azure-stack-hub-diagnostic-logs-now"></a>立即发送 Azure Stack Hub 诊断日志

在请求支持之前，Azure Stack 操作员可以使用管理员门户或 PowerShell 将诊断日志按需发送到 Azure 支持。 如果 Azure Stack Hub 已连接到 Azure，建议使用管理员门户中的“立即发送日志”选项，因为这是直接将日志发送到 Azure 的最简单方法。 如果门户不可用，则操作员应改为[使用 Send-AzureStackDiagnosticLog 立即发送日志](./azure-stack-configure-on-demand-diagnostic-log-collection-powershell.md?view=azs-2002)。 

如果已断开与 Internet 的连接，或者只想在本地保存日志，请使用 [Get-AzureStackLog](azure-stack-get-azurestacklog.md) 方法发送日志。 以下流程图显示了每种情况下用于发送诊断日志的选项。 

![流程图，显示如何将日志立即发送到 Microsoft](media/azure-stack-help-and-support/send-logs-now-flowchart.png)

>[!NOTE]
>可以通过[主动收集诊断日志](./azure-stack-configure-automatic-diagnostic-log-collection.md?view=azs-2002)来简化故障排除过程，这是一种替代按需收集日志的方法。 如果需要调查系统运行状况，则会在使用 Azure 支持建立案例之前自动上传日志进行分析。 如果启用了主动收集日志功能，则当日志收集正在进行时，会显示“帮助和支持”。 如果在进行主动日志收集时单击“立即发送日志”从特定时间收集日志，则会在主动日志收集完成后开始按需收集。

指定日志收集的开始时间和结束时间，然后单击“收集并上传”。 

![“立即发送日志”选项的屏幕截图](media/azure-stack-help-and-support/send-logs-now.png)


## <a name="save-logs-locally"></a>在本地保存日志

当 Azure Stack Hub 与 Azure 断开连接时，可以将日志保存到本地 SMB 共享。 在“设置”边栏选项卡中，输入具有共享写入权限的路径、用户名和密码。 在支持案例期间，Azure 支持会提供详细步骤来说明如何传输这些本地日志。 如果管理员门户不可用，则可以使用 [Get-AzureStackLog](azure-stack-get-azurestacklog.md) 在本地保存日志。

![诊断日志收集选项的屏幕截图](media/azure-stack-help-and-support/save-logs-locally.png)

## <a name="use-help-and-support-to-collect-diagnostic-logs-on-demand"></a>使用“帮助和支持”按需收集诊断日志

为了排查问题，Azure 支持可能会请求 Azure Stack Hub 操作员根据需要收集上周特定时间范围的诊断日志。 在这种情况下，Azure 支持会为操作员提供一个 SAS URL 来上传收集的内容。 使用以下步骤，通过 Azure 支持提供的 SAS URL 配置按需日志收集：

1. 打开“帮助 + 支持”>“日志收集”>“立即收集日志”。 
1. 从过去七天选择 1-4 小时滑动窗口。 
1. 选择本地时区。
1. 输入 Azure 支持提供的 SAS URL。

   ![按需日志收集的屏幕截图](media/azure-stack-automatic-log-collection/collect-logs-now.png)

>[!NOTE]
>如果启用了诊断日志自动收集，“帮助和支持”会显示何时在进行日志收集。 如果单击“立即收集日志”以收集特定时间的日志，而此时系统正在进行自动日志收集，则当自动日志收集完成后，按需收集就会开始。 


## <a name="next-steps"></a>后续步骤

[使用特权终结点 (PEP) 发送 Azure Stack Hub 诊断日志](./azure-stack-configure-on-demand-diagnostic-log-collection-powershell.md?view=azs-2002)
