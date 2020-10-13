---
title: 启用卷加密、重复数据删除和压缩 - Azure Stack HCI
description: 本主题介绍如何使用 Windows Admin Center 在 Azure Stack HCI 中使用卷加密、重复数据删除和压缩。
author: WenJason
ms.author: v-jay
ms.topic: how-to
ms.service: azure-stack
origin.date: 09/03/2020
ms.date: 10/12/2020
ms.openlocfilehash: 86130d5d2f898a6b2af8de5fe12c479ffb164ee9
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91451230"
---
# <a name="enable-volume-encryption-deduplication-and-compression-in-azure-stack-hci"></a>在 Azure Stack HCI 中启用卷加密、重复数据删除和压缩

> 适用于：Azure Stack HCI 版本 20H2；Windows Server 2019

本主题介绍如何使用 Windows Admin Center 在 Azure Stack HCI 中对卷启用 BitLocker 加密。 还介绍如何对卷启用重复数据删除和压缩。 若要了解如何创建卷，请参阅[创建卷](create-volumes.md)。

## <a name="turn-on-bitlocker-to-protect-volumes"></a>打开 BitLocker 以保护卷
若要在 Windows Admin Center 中打开 BitLocker：

1. 连接到存储空间直通群集，然后在“工具”窗格上，选择“卷” 。
1. 在“卷”页上，选择“盘存”选项卡，然后在“可选功能”下，打开“加密(BitLocker)”切换开关   。

    :::image type="content" source="media/volume-encryption-deduplication/bitlocker-toggle-switch.png" alt-text="用于启用 BitLocker 的切换开关":::

1. 在“加密 (BitLocker)”弹出项中，选择“开始”，然后在“启用加密”页上，提供凭据以完成工作流  。

>[!NOTE]
> 如果显示“先安装 BitLocker 功能”弹出项，请按照其说明在群集中的每个服务器上安装该功能，然后重新启动服务器。

## <a name="turn-on-deduplication-and-compression"></a>启用重复数据删除和压缩
重复数据删除和压缩是根据每个卷进行管理的。 重复数据删除和压缩使用后处理模型，这意味着，在该功能运行之前，你看不到节省的空间。 该功能在运行时会处理所有文件，甚至包括以前就已存在的文件。

若要在 Windows Admin Center 中对卷打开重复数据删除和压缩：

1. 连接到存储空间直通群集，然后在“工具”窗格上，选择“卷” 。
1. 在“卷”页上选择“库存”选项卡。 
1. 在卷列表中，选择要管理的卷的名称。
1. 在卷详细信息页上，打开“重复数据删除和压缩”切换开关。
1. 在“启用重复数据删除”窗格中选择重复数据删除模式。

    Windows Admin Center 可让你针对不同的工作负荷选择现成的配置文件，而无需进行复杂的设置。 如果你不确定要如何设置，请使用默认设置。

1. 选择“启用重复数据删除”。

启用卷加密对卷性能的影响较小（通常低于 10%），但影响因硬件和工作负载而异。 重复数据删除和压缩也会对性能产生影响 — 有关详细信息，请参阅[确定适合进行重复数据删除的工作负载](https://docs.microsoft.com/windows-server/storage/data-deduplication/install-enable#enable-dedup-candidate-workloads)。

<!---Add info on greyed out ReFS option? --->

大功告成！ 根据需要重复操作以保护卷中的数据。

## <a name="next-steps"></a>后续步骤
有关相关主题和其他存储管理任务，另请参阅：

- [存储空间直通概述](https://docs.microsoft.com/windows-server/storage/storage-spaces/storage-spaces-direct-overview)
- [规划卷](../concepts/plan-volumes.md)
- [扩展卷](extend-volumes.md)
- [删除卷](delete-volumes.md)