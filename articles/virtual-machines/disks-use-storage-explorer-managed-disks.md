---
title: 使用 Azure 存储资源管理器管理 Azure 托管磁盘
description: 了解如何使用 Azure 存储资源管理器跨区域上传、下载和迁移 Azure 托管磁盘并创建托管磁盘的快照。
origin.date: 09/25/2019
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 09/07/2020
ms.author: v-yeche
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: 20424f996e7343fbc3a185c04bf11f6a1de448db
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103937"
---
<!--Verified successfully from renamed articles-->
# <a name="use-azure-storage-explorer-to-manage-azure-managed-disks"></a>使用 Azure 存储资源管理器管理 Azure 托管磁盘

存储资源管理器 1.10.0 可让用户上传、下载和复制托管磁盘，以及创建快照。 由于具有这些附加功能，存储资源管理器使你能够将数据从本地迁移到 Azure，并可以跨 Azure 区域迁移数据。

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：
- Azure 订阅
- 一个或多个 Azure 托管磁盘
- 最新版本的 [Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)

## <a name="connect-to-an-azure-subscription"></a>连接到 Azure 订阅

如果存储资源管理器未连接到 Azure，则将无法使用它来管理资源。 本部分介绍如何将存储资源管理器连接到 Azure 帐户，以便可以使用它来管理资源。

1. 启动 Azure 存储资源管理器并单击左侧的 **插件** 图标。

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/plug-in-icon.png" alt-text="单击插件图标":::

1. 选择“添加 Azure 帐户”，然后单击“下一步”。  

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/connect-to-azure.png" alt-text="单击插件图标":::

1. 在“Azure 登录”对话框中输入 Azure 凭据。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/sign-in.png" alt-text="单击插件图标":::

1. 从列表中选择你的订阅，然后单击“应用”。

    选择订阅

## <a name="upload-a-managed-disk-from-an-on-prem-vhd"></a>从本地 VHD 上传托管磁盘

1. 在左窗格中，展开“磁盘”并选择要将磁盘上传到的资源组。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg1.png" alt-text="单击插件图标":::

1. 选择“上传”。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/upload-button.png" alt-text="单击插件图标":::

1. 在“上传 VHD”中，指定源 VHD、磁盘名称、OS 类型、要将磁盘上传到的区域，以及帐户类型。  

    <!--Not Available on In some regions Availability zones are supported, for those regions you can select a zone of your choice.-->

1. 选择“创建”开始上传磁盘。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/upload-vhd-dialog.png" alt-text="单击插件图标":::

1. 现在，上传状态会显示在“活动”中。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/activity-uploading.png" alt-text="单击插件图标":::

1. 如果上传已完成但右窗格中未显示该磁盘，请选择“刷新”。 

## <a name="download-a-managed-disk"></a>下载托管磁盘

以下步骤说明如何将托管磁盘下载到本地 VHD。 只能下载状态为“未附加”的磁盘，而无法下载状态为“已附加”的磁盘。  

1. 在左窗格中展开“磁盘”（如果尚未展开），并选择要从中下载磁盘的资源组。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg1.png" alt-text="单击插件图标":::

1. 在右窗格中选择要下载的磁盘。
1. 选择“下载”，然后选择磁盘的保存位置。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/download-button.png" alt-text="单击插件图标":::

1. 选择“保存”，随即会开始下载磁盘。  下载状态将显示在“活动”中。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/activity-downloading.png" alt-text="单击插件图标":::

## <a name="copy-a-managed-disk"></a>复制托管磁盘

使用存储资源管理器可以在区域内部或跨区域复制托管磁盘。 若要复制磁盘：

1. 在左侧的“磁盘”下拉列表中，选择包含要复制的磁盘的资源组。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg1.png" alt-text="单击插件图标":::

1. 在右窗格中选择要复制的磁盘，然后选择“复制”。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/copy-button.png" alt-text="单击插件图标":::

1. 在左窗格中，选择要将磁盘粘贴到的资源组。

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg2.png" alt-text="单击插件图标":::

1. 在右窗格中选择“粘贴”。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/paste-button.png" alt-text="单击插件图标":::

1. 在“粘贴磁盘”对话框中填写值。 

    <!--Not Available on You can also specify an Availability zone in supported regions.-->

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/paste-disk-dialog.png" alt-text="单击插件图标":::

1. 选择“粘贴”，磁盘随即开始复制，复制状态会显示在“活动”中。  

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/activity-copying.png" alt-text="单击插件图标":::

## <a name="create-a-snapshot"></a>创建快照

1. 在左侧的“磁盘”下拉列表中，选择要创建快照的磁盘所在的资源组。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg1.png" alt-text="单击插件图标":::

1. 在右侧选择要创建快照的磁盘，然后选择“创建快照”。 

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/create-snapshot-button.png" alt-text="单击插件图标":::

1. 在“创建快照”中，指定快照名称，以及要在其中创建快照的资源组。  然后选择“创建”  。

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/create-snapshot-dialog.png" alt-text="单击插件图标":::

1. 创建快照后，可以在“活动”中选择“在门户中打开”，以便在 Azure 门户中查看快照。  

    :::image type="content" source="media/disks-upload-vhd-to-managed-disk-storage-explorer/open-in-portal.png" alt-text="单击插件图标":::

## <a name="next-steps"></a>后续步骤

了解如何[使用 Azure 门户基于 VHD 创建 VM](https://docs.azure.cn/virtual-machines/windows/create-vm-specialized-portal)。

了解如何[使用 Azure 门户将托管数据磁盘附加到 Windows VM](https://docs.azure.cn/virtual-machines/windows/attach-managed-disk-portal)。

<!-- Update_Description: new article about disks use storage explorer managed disks -->
<!--NEW.date: 09/07/2020-->