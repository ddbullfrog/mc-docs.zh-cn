---
title: 在应用默认域控制器策略时，VM 无响应
titlesuffix: Azure Virtual Machines
description: 本文介绍了如何解决默认域控制器策略阻止 Azure VM 启动的问题。
services: virtual-machines-windows, azure-resource-manager
manager: dcscontentpm
tags: azure-resource-manager
ms.assetid: 34e6b765-3496-46a1-b7d7-6def00884394
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
origin.date: 09/08/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: no
ms.testdate: 10/19/2020
ms.author: v-yeche
ms.openlocfilehash: 5734461d902f38846ae973cfeb6145c3e5baf338
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105931"
---
<!--Verified Successfully-->
# <a name="vm-is-unresponsive-while-applying-default-domain-controllers-policy"></a>在应用默认域控制器策略时，VM 无响应

本文介绍了如何解决默认域控制器策略阻止 Azure 虚拟机 (VM) 启动的问题。

## <a name="symptom"></a>症状

使用[启动诊断](./boot-diagnostics.md)查看 VM 的屏幕截图时，你将看到屏幕截图显示启动时 OS 无响应，并显示消息“默认域控制器策略”。

  :::image type="content" source="./media/vm-unresponsive-domain-controllers-policy/1-default-domain-controllers-policy.png" alt-text="图 1 显示 OS 卡住，出现消息“默认域控制器策略”":::

## <a name="cause"></a>原因

出现此问题可能是因为最近对默认域控制器策略进行了更改。 否则，需要执行内存转储文件分析来确定问题的根本原因。

## <a name="solution"></a>解决方案

如果你最近更改了默认域控制器策略，则可能想要撤消这些更改以解决此问题。 如果你不确定导致此问题的原因，请收集内存转储，然后提交支持工单。

### <a name="collect-the-memory-dump-file"></a>收集内存转储文件

若要解决此问题，应先收集故障的内存转储文件，然后使用此内存转储文件联系支持部门。 若要收集转储文件，请执行以下步骤：

### <a name="attach-the-os-disk-to-a-new-repair-vm"></a>将 OS 磁盘附加到新的修复 VM

1. 使用 [VM 修复命令](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md)的步骤 1-3 来准备一个修复 VM。

1. 使用远程桌面连接来连接到修复 VM。

### <a name="locate-the-dump-file-and-submit-a-support-ticket"></a>找到转储文件并提交支持票证

1. 在修复 VM 上，转到附加的 OS 磁盘中的 Windows 文件夹。 如果分配给附加 OS 磁盘的驱动器号为 `F`，则需转到 `F:\Windows`。

1. 找到 memory.dmp 文件，然后使用该内存转储文件[提交支持票证](https://support.azure.cn/support/support-azure/)。

<!--Not Available on [non-maskable interrupt (NMI) calls in serial console](./serial-console-windows.md#use-the-serial-console-for-nmi-calls)-->

<!-- Update_Description: update meta properties, wording update, update link -->