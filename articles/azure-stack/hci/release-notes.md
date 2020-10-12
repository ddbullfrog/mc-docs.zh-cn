---
title: Azure Stack HCI 发行说明
description: Azure Stack HCI，版本 20H2 发行说明。
author: WenJason
ms.author: v-jay
ms.topic: conceptual
ms.service: azure-stack
ms.subservice: azure-stack-hci
origin.date: 09/17/2020
ms.date: 10/12/2020
ms.openlocfilehash: 0bf7715f93926ae3aaca3b340f7a4b35e3dce7ea
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91437608"
---
# <a name="release-notes-for-azure-stack-hci-public-preview"></a>Azure Stack HCI 公共预览版发行说明

> 适用于：Azure Stack HCI，版本 20H2

本文介绍 Azure Stack HCI 公共预览版更新包的内容。

## <a name="september-17-2020-preview-update-kb4577629"></a>2020 年 9 月 17 日预览版更新 (KB4577629)

此更新包括最新版 Azure Stack HCI 的改进和修复。

### <a name="improvements-and-fixes"></a>改进和修复
此非安全更新包括质量改进。 关键更改包括：
- 解决了下面的问题：流过多路复用器的软件负载均衡器 (SLB) 流量可能会重定向到可能导致应用程序连接失败的另一个主机。

### <a name="known-issues-in-this-update"></a>此更新中的已知问题
Azure 目前未了解到任何有关此更新的问题。

### <a name="how-to-get-this-update"></a>如何获取此更新
[Azure Stack HCI 预览版](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)的 2020 年 9 月 17 日安全更新 (KB4577629) 通过 Windows 更新交付。 若要在 Azure Stack HCI 群集上安装该安全更新，请参阅[更新 Azure Stack HCI 群集](manage/update-cluster.md)。

### <a name="file-information"></a>文件信息
有关此更新（OS 内部版本 17784.1259）中提供的文件列表，请下载[累积更新 4577629 的文件信息](https://download.microsoft.com/download/9/1/a/91addcbb-2b36-408c-ab88-736de42edb98/4577629.csv)

   > [!NOTE]
   > 某些文件在 CSV 文件的“文件版本”列中错误地显示为“不适用”。 当使用某些第三方扫描检测工具验证生成时，这可能会导致假正或误报。

## <a name="september-8-2020-security-update-kb4577470"></a>2020 年 9 月 8 日安全更新 (KB4577470)

此更新包括最新版 Azure Stack HCI 的改进和修复。

### <a name="improvements-and-fixes"></a>改进和修复
此更新包含对内部 OS 功能的各种安全改进。 此版本没有其他已知问题。

有关已解决的安全漏洞的详细信息，请参阅[安全更新指南](https://portal.msrc.microsoft.com/security-guidance)。

### <a name="known-issues-in-this-update"></a>此更新中的已知问题
Azure 目前未了解到任何有关此更新的问题。

### <a name="how-to-get-this-update"></a>如何获取此更新
[Azure Stack HCI 预览版](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)的 2020 年 9 月 8 日安全更新 (KB4577470) 通过 Windows 更新交付。 若要在 Azure Stack HCI 群集上安装该安全更新，请参阅[更新 Azure Stack HCI 群集](manage/update-cluster.md)。

### <a name="file-information"></a>文件信息
有关此更新（OS 内部版本 17784.1226）中提供的文件列表，请下载[累积更新 4577470 的文件信息](https://download.microsoft.com/download/3/c/4/3c468525-5867-4cc3-8d34-dba88989adab/4577470.csv)。

   > [!NOTE]
   > 某些文件在 CSV 文件的“文件版本”列中错误地显示为“不适用”。 当使用某些第三方扫描检测工具验证生成时，这可能会导致假正或误报。

## <a name="september-8-2020-servicing-stack-update-kb4577558"></a>2020 年 9 月 8 日服务堆栈更新 (KB4577558)

此更新包括对最新版 Azure Stack HCI 的质量改进。

### <a name="improvements-and-fixes"></a>改进和修复
此更新对服务堆栈（安装更新的组件）进行质量改进。 服务堆栈更新 (SSU) 确保你具有强大可靠的服务堆栈，以便你的设备可以接收并安装 Microsoft 更新。

### <a name="how-to-get-this-update"></a>如何获取此更新
[Azure Stack HCI 预览版](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)的 2020 年 9 月 8 日服务堆栈更新 (KB4577558) 通过 Windows 更新交付。 若要在 Azure Stack HCI 群集上安装该安全更新，请参阅[更新 Azure Stack HCI 群集](manage/update-cluster.md)。

### <a name="restart-information"></a>“重启”信息 
应用此更新后，无需重启计算机。

### <a name="removal-information"></a>“删除”信息
服务堆栈更新 (SSU) 对安装更新的方式进行了更改，并且无法从设备中卸载。

### <a name="file-information"></a>文件信息
有关此更新（OS 内部版本 17784.1220）中提供的文件列表，请下载[累积更新 4577558 的文件信息](https://download.microsoft.com/download/8/f/6/8f612a9b-cb4e-4832-9397-156760848592/4577558.csv)。

### <a name="references"></a>参考

有关 SSU 的信息，请查看以下文章：

- [服务堆栈更新](https://docs.microsoft.com/windows/deployment/update/servicing-stack-updates)
- [服务堆栈更新 (SSU)：常见问题解答](https://support.microsoft.com/help/4535697)

了解 Microsoft 用于描述软件更新的[术语](https://support.microsoft.com/help/824684)。

## <a name="august-11-2020-security-update-kb4574585"></a>2020 年 8 月 11 日安全更新 (KB4574585)

此更新包括最新版 Azure Stack HCI 的改进和修复。

### <a name="improvements-and-fixes"></a>改进和修复
此更新包含对内部 OS 功能的各种安全改进。 此版本没有其他已知问题。

有关已解决的安全漏洞的详细信息，请参阅[安全更新指南](https://portal.msrc.microsoft.com/security-guidance)。

### <a name="known-issues-in-this-update"></a>此更新中的已知问题
Microsoft 目前未了解到任何有关此更新的问题。

### <a name="how-to-get-this-update"></a>如何获取此更新
[Azure Stack HCI 预览版](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)的 2020 年 8 月 11 日安全更新 (KB4574585) 通过 Windows 更新交付。 若要在 Azure Stack HCI 群集上安装该安全更新，请参阅[更新 Azure Stack HCI 群集](manage/update-cluster.md)。

### <a name="file-information"></a>文件信息
有关此更新（OS 内部版本 17784.1162）中提供的文件列表，请下载[累积更新 4574585 的文件信息](https://download.microsoft.com/download/7/f/4/7f451def-76c5-4cc0-9929-0c5efeb27d2f/4574585.csv)。

   > [!NOTE]
   > 某些文件在 CSV 文件的“文件版本”列中错误地显示为“不适用”。 当使用某些第三方扫描检测工具验证生成时，这可能会导致假正或误报。
