---
title: OS 启动 - 计算机意外重启或遇到意外错误
description: 本文提供的步骤可用于解决 VM 在安装 Windows 时遇到意外重启或错误的问题。
services: virtual-machines-windows, azure-resource-manager
manager: dcscontentpm
tags: azure-resource-manager
ms.assetid: 1d249a4e-71ba-475d-8461-31ff13e57811
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
origin.date: 06/22/2020
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: 32c7d4023cee9857947eac240d1cf3c288994049
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104157"
---
# <a name="os-start-up---computer-restarted-unexpectedly-or-encountered-an-unexpected-error"></a>OS 启动 - 计算机意外重启或遇到意外错误

本文提供的步骤可用于解决虚拟机 (VM) 在安装 Windows 时遇到意外重启或错误的问题。

## <a name="symptom"></a>症状

使用[启动诊断](./boot-diagnostics.md)查看 VM 的屏幕截图时，可看到屏幕截图显示 Windows 安装失败，出现以下错误：

**计算机意外重启或遇到意外错误。Windows 安装无法继续。要安装 Windows，请单击“确定”重启计算机，然后重新开始安装。**

![进行 Windows 安装时出错：计算机意外重启或遇到意外错误。 Windows 安装无法继续。 要安装 Windows，请单击“确定”重启计算机，然后重新开始安装。](./media/unexpected-restart-error-during-vm-boot/1.png)

![Windows 安装程序启动服务时出错：计算机意外重启或遇到意外错误。 Windows 安装无法继续。 要安装 Windows，请单击“确定”重启计算机，然后重新开始安装。](./media/unexpected-restart-error-during-vm-boot/2.png)

## <a name="cause"></a>原因

虚拟机在尝试初次启动[通用映像](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation)，但因正在处理的自定义应答文件 (unattend.xml) 而遇到了问题。 Azure 中不支持自定义应答文件。 

应答文件是一种特殊的 XML 文件，其中包含要在安装 Windows Server 操作系统的过程中自动完成的配置设置的设置定义和值。 配置选项包括有关以下内容的说明：如何对磁盘进行分区、在何处查找要安装的 Windows 映像、要应用的产品密钥，以及要运行的其他命令。

Azure 中不支持自定义应答文件。 如果使用 `sysprep /unattend:<your file's name>` 选项指定了自定义 Unattend.xml 文件，则会出现此错误。

在 Azure 中，使用 Sysprep.exe 中的“进入系统全新安装体验 (OOBE)”选项，或使用 `sysprep /oobe`，而不是 Unattend.xml 文件 。

此问题通常是在将 Sysprep.exe 与本地 VM 一起使用以将通用 VM 上传到 Azure 时发生。 在这种情况下，你可能还对如何正确上传通用 VM 感兴趣。

## <a name="solution"></a>解决方案

### <a name="replace-unattended-answer-file-option"></a>替换无人参与应答文件选项

当已准备好在 Azure 中使用某个映像，但该映像使用了 Azure 不支持的自定义应答文件，并且你使用的 SYSPREP 带有类似以下命令的标志时，会发生这种情况：

`sysprep /oobe /generalize /unattend:<NameOfYourAnswerFile.XML> /shutdown`

- 在前面的命令中，将 `<NameOfYourAnswerFile.XML>` 替换为你的文件名称。

要修复此问题，请按照[有关准备/捕获映像的 Azure 指南](../windows/upload-generalized-managed.md)操作，并准备新的通用映像。 在 sysprep 期间，请勿使用 `/unattend:<answerfile>` 标志。 请改为仅使用下面的标志：

`sysprep /oobe /generalize /shutdown`

- 全新安装体验 (OOBE) 是 Azure VM 支持的设置。

你还可以通过选择下面显示的选项，使用“系统准备工具 GUI”来完成通过上述命令完成的相同任务：

- 进入全新安装体验
- 通用化
- Shutdown

:::image type="content" source="./media/unexpected-restart-error-during-vm-boot/3.png" alt-text="“系统准备工具”窗口，其中选中了“OOBE”、“通用化”和“关闭”选项。":::

<!-- Update_Description: update meta properties, wording update, update link -->