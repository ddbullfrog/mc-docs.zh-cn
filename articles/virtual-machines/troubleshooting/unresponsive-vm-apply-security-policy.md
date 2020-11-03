---
title: Azure VM 在将安全策略应用于系统时没有响应
description: 本文提供了一些步骤来解决在将安全策略应用于 Azure VM 中的系统时，VM 没有响应且加载屏幕卡住的问题。
services: virtual-machines-windows
author: rockboyfor
manager: digimobile
tags: azure-resource-manager
ms.assetid: a97393c3-351d-4324-867d-9329e31b5629
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
origin.date: 06/15/2020
ms.date: 07/27/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: f858d521bf9f0dc2225459a87b1f8f27dd2e5e42
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106209"
---
# <a name="azure-vm-is-unresponsive-while-applying-security-policy-to-the-system"></a>Azure VM 在将安全策略应用于系统时没有响应

本文提供了一些步骤来解决在 Azure VM 中应用安全策略时，操作系统挂起且无响应的问题。

## <a name="symptoms"></a>症状

使用[启动诊断](boot-diagnostics.md)查看 VM 的屏幕截图时，你将看到屏幕截图显示启动时操作系统卡住，并显示以下消息：

> “将安全策略应用于系统”。

:::image type="content" source="media/unresponsive-vm-apply-security-policy/apply-policy.png" alt-text="Windows Server 2012 R2 启动屏幕卡住的屏幕截图。":::

:::image type="content" source="media/unresponsive-vm-apply-security-policy/apply-policy-2.png" alt-text="Windows Server 2012 R2 启动屏幕卡住的屏幕截图。":::

## <a name="cause"></a>原因

此问题的可能原因有很多。 在执行内存转储分析之前，你将无法知道该源。

## <a name="resolution"></a>解决方法

### <a name="process-overview"></a>流程概述

1. [创建和访问修复 VM](#create-and-access-a-repair-vm)
    
    <!--Not Available on 2. [Enable Serial Console and Memory Dump Collection](#enable-serial-console-and-memory-dump-collection)-->
    
3. [重新生成 VM](#rebuild-the-vm)
4. [收集内存转储文件](#collect-the-memory-dump-file)

### <a name="create-and-access-a-repair-vm"></a>创建并访问修复 VM

1. 使用 [VM 修复命令的步骤 1-3](repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) 来准备一个修复 VM。
2. 使用远程桌面连接来连接到修复 VM。

<!--Not Available on ### Enable Serial Console and Memory Dump Collection-->


### <a name="rebuild-the-vm"></a>重新生成 VM

使用 [VM 修复命令的步骤 5](repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) 重新装配 VM。

### <a name="collect-the-memory-dump-file"></a>收集内存转储文件

若要解决此问题，需先收集故障的内存转储文件，然后使用此内存转储文件联系支持部门。 若要收集转储文件，请执行以下步骤：

1. 将 OS 磁盘附加到新的修复 VM：

    - 使用 [VM 修复命令的步骤 1-3](repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) 来准备一个新的修复 VM。
    - 使用远程桌面连接来连接到修复 VM。

2. 找到转储文件并提交支持票证：

    - 在修复 VM 上，转到附加的 OS 磁盘中的 Windows 文件夹。 如果分配给附加 OS 磁盘的驱动器号为 `F`，则需转到 `F:\Windows`。
    - 找到 memory.dmp 文件，然后使用该内存转储文件[提交支持票证](https://support.azure.cn/support/support-azure/)。
    
    <!--Not Available on [non-maskable interrupt (NMI) calls in serial console](serial-console-windows.md#use-the-serial-console-for-nmi-calls)-->
    
## <a name="next-steps"></a>后续步骤

如果在应用本地用户和组策略时遇到问题，请参阅[应用组策略本地用户和组策略时，VM 无响应](unresponsive-vm-apply-group-policy.md)

<!-- Update_Description: new article about unresponsive vm apply security policy -->
<!--NEW.date: 07/27/2020-->