---
title: 复制并粘贴到虚拟机以及从中进行复制和粘贴 - Azure Bastion
description: 本文介绍如何使用 Bastion 复制并粘贴到 Azure VM 以及从中进行复制和粘贴。
services: bastion
ms.service: bastion
ms.topic: how-to
origin.date: 05/04/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: ed4f68f9c21f335a86ed84a8165430bcd891cc97
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104822"
---
<!--Verified successfully on 09/07/2020-->
# <a name="copy-and-paste-to-a-virtual-machine-azure-bastion"></a>复制并粘贴到虚拟机：Azure Bastion

本文可帮助你在使用 Azure Bastion 时将内容复制并粘贴到虚拟机，以及从中进行复制和粘贴。 使用 VM 之前，请确保已按照[创建 Bastion 主机](./tutorial-create-host-portal.md)的步骤进行操作。 然后，连接到要通过 [RDP](bastion-connect-vm-rdp.md) 或 [SSH](bastion-connect-vm-ssh.md) 使用的 VM。

对于支持高级剪贴板 API 访问的浏览器，可以使用在本地设备上的应用程序之间进行复制和粘贴的相同方式在本地设备与远程会话之间复制和粘贴文本。 对于其他浏览器，可以使用 Bastion 剪贴板访问工具面板。

>[!NOTE]
>目前仅支持文本复制/粘贴。
>

:::image type="content" source="./media/bastion-vm-manage/allow.png" alt-text="允许剪贴板":::

仅支持文本复制/粘贴。 对于直接复制和粘贴，浏览器可能会在 Bastion 会话进行初始化时提示你它需要访问剪贴板。 允许网页访问剪贴板。

<a name="to"></a>
## <a name="copy-to-a-remote-session"></a>复制到远程会话

使用 [Azure 门户](https://portal.azure.cn)连接到虚拟机后，请完成以下步骤：

1. 将本地设备中的文本/内容复制到本地剪贴板。
1. 在远程会话期间，通过选择两个箭头启动 Bastion 剪贴板访问工具面板。 这些箭头位于会话的左中位置。

    :::image type="content" source="./media/bastion-vm-manage/left.png" alt-text="屏幕截图显示了在窗口左侧突出显示的工具面板的启动箭头。":::

    :::image type="content" source="./media/bastion-vm-manage/clipboard.png" alt-text="屏幕截图显示了在 Bastion 中复制的文本的剪贴板。":::
1. 通常，已复制的文本会自动显示在 Bastion 复制粘贴面板上。 如果文本不在那里，则将文本粘贴到面板上的文本区域中。
1. 文本位于文本区域中后，可以将其粘贴到远程会话。

    :::image type="content" source="./media/bastion-vm-manage/local.png" alt-text="屏幕截图显示了突出显示的复制/粘贴按钮和复制到远程会话的示例文本字符串。":::

<a name="from"></a>
## <a name="copy-from-a-remote-session"></a>从远程会话复制

使用 [Azure 门户](https://portal.azure.cn)连接到虚拟机后，请完成以下步骤：

1. 将远程会话中的文本/内容复制到远程剪贴板（使用 Ctrl-C）。

    :::image type="content" source="./media/bastion-vm-manage/remote.png" alt-text="工具面板":::
1. 在远程会话期间，通过选择两个箭头启动 Bastion 剪贴板访问工具面板。 这些箭头位于会话的左中位置。

    :::image type="content" source="./media/bastion-vm-manage/clipboard2.png" alt-text="剪贴板":::
1. 通常，已复制的文本会自动显示在 Bastion 复制粘贴面板上。 如果文本不在那里，则将文本粘贴到面板上的文本区域中。
1. 文本位于文本区域中后，可以将其粘贴到本地设备。

    :::image type="content" source="./media/bastion-vm-manage/local2.png" alt-text="粘贴":::

## <a name="next-steps"></a>后续步骤

阅读 [Bastion 常见问题解答](bastion-faq.md)。

<!-- Update_Description: update meta properties, wording update, update link -->