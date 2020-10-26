---
title: 排查 Windows 无法完成系统配置的问题
titlesuffix: Azure Virtual Machines
description: 本文介绍了如何解决 Sysprep 进程阻止 Azure VM 启动的问题。
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 5fc57a8f-5a0c-4b5f-beef-75eecb4d310d
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
origin.date: 09/09/2020
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: yes
ms.testdate: 10/19/2020
ms.author: v-yeche
ms.openlocfilehash: b703e85ffc95ad16f0b9c427212bc00ee4d8329d
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128750"
---
<!--Verified Successfully-->
# <a name="troubleshoot-windows-could-not-finish-configuring-the-system"></a>排查 Windows 无法完成系统配置的问题

本文介绍了如何解决 Sysprep 进程阻止 Azure 虚拟机 (VM) 启动的问题。

## <a name="symptom"></a>症状

使用[启动诊断](/virtual-machines/troubleshooting/boot-diagnostics)查看 VM 的屏幕截图时，你会看到 Windows 安装程序启动服务时，该屏幕截图显示“安装 Windows”错误。 此错误将显示消息：

`Windows could not finish configuring the system. To attempt to resume configuration, restart the computer. Setup is starting services`

    ![Figure 1 displays an Install Windows error with the message: "Windows could not finish configuring the system. To attempt to resume configuration, restart the computer. Setup is starting services"](./media/windows-could-not-configure-system/1-windows-error-configure.png)

## <a name="cause"></a>原因

当操作系统 (OS) 无法完成 [Sysprep 进程](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep-process-overview)时，则会导致此错误。 尝试初次启动通用 VM 时，将发生此错误。 如果遇到此问题，请重新创建通用映像，因为该映像处于不可部署状态且无法恢复。

## <a name="solution"></a>解决方案

要修复此问题，请按照[有关准备/捕获映像的 Azure 指南](/virtual-machines/windows/upload-generalized-managed)操作，并准备新的通用映像。

<!-- Update_Description: new article about windows could not configure system -->
<!--NEW.date: 10/19/2020-->