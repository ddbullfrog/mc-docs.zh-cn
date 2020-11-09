---
title: 更改 Azure 托管磁盘的性能
description: 了解托管磁盘的性能层，并了解如何更改现有托管磁盘的性能层。
ms.service: virtual-machines
ms.topic: how-to
origin.date: 09/24/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 11/02/2020
ms.author: v-yeche
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 0ad2a0d0c07c768e9ea05bee195bdec0236344d1
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106712"
---
<!--Verified Successfully-->
# <a name="performance-tiers-for-managed-disks-preview"></a>托管磁盘的性能层（预览）

Azure 磁盘存储目前提供内置突发功能，以提供更高的性能来处理短期的意外流量。 高级 SSD 在不增加实际磁盘大小的情况下，可灵活地提高磁盘性能。 此功能使你可以匹配工作负载性能需求并降低成本。 

> [!NOTE]
> 此功能目前处于预览状态。 

此功能非常适合于那些暂时需要持续较高性能级别的事件，例如假日购物、性能测试或运行训练环境。 为了应对这些事件，你可根据需要使用级别较高的性能层。 当你不再需要额外性能时，可返回到原始层。

## <a name="how-it-works"></a>工作原理

首次部署或预配磁盘时，该磁盘的基线性能层将根据预配的磁盘大小进行设置。 可使用级别较高的性能层来满足更高的需求。 当你不再需要该性能级别时，可返回到初始基线性能层。

你的帐单会随层级的变化而变化。 例如，如果你预配了 P10 磁盘 (128 GiB)，基线性能层将设置为 P10（500 IOPS 和 100 MBps）。 将按 P10 费率计费。 可以升级该层，以匹配 P50（7500 IOPS 和 250 MBps）的性能，而不增加磁盘大小。 在升级期间，将按 P50 费率计费。 如果不再需要更高的性能，可返回到 P10 层。 磁盘将再次按 P10 费率计费。

| 磁盘大小 | 基线性能层 | 可升级到 |
|----------------|-----|-------------------------------------|
| 4 GiB | P1 | P2、P3、P4、P6、P10、P15、P20、P30、P40、P50 |
| 8 GiB | P2 | P3、P4、P6、P10、P15、P20、P30、P40、P50 |
| 16 GiB | P3 | P4、P6、P10、P15、P20、P30、P40、P50 | 
| 32 GiB | P4 | P6、P10、P15、P20、P30、P40、P50 |
| 64 GiB | P6 | P10、P15、P20、P30、P40、P50 |
| 128 GiB | P10 | P15、P20、P30、P40、P50 |
| 256 GiB | P15 | P20、P30、P40、P50 |
| 512 GiB | P20 | P30、P40、P50 |
| 1 TiB | P30 | P40、P50 |
| 2 TiB | P40 | P50 |
| 4 TiB | P50 | 无 |
| 8 TiB | P60 |  P70、P80 |
| 16 TiB | P70 | P80 |
| 32 TiB | P80 | 无 |

有关计费信息，请参阅[托管磁盘定价](https://www.azure.cn/pricing/details/storage/managed-disks/)。

## <a name="restrictions"></a>限制

- 目前仅高级 SSD 支持此功能。
- 必须先从正在运行的 VM 中拆离磁盘，然后才能更改磁盘层级。
- P60、P70 和 P80 性能层的使用仅限于 4096 GiB 或更大的磁盘。
- 磁盘的性能层每 24 小时只能更改一次。

## <a name="regional-availability"></a>区域可用性

调整托管磁盘的性能层的功能目前仅在中国东部 2、中国东部、中国北部、澳大利亚东南部地区的高级 SSD 上可用。

## <a name="create-an-empty-data-disk-with-a-tier-higher-than-the-baseline-tier"></a>创建一个层级高于基线层的空数据磁盘

```azurecli
subscriptionId=<yourSubscriptionIDHere>
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
diskSize=<yourDiskSizeHere>
performanceTier=<yourDesiredPerformanceTier>
region=chinaeast

az login

az account set --subscription $subscriptionId

az disk create -n $diskName -g $resourceGroupName -l $region --sku Premium_LRS --size-gb $diskSize --tier $performanceTier
```
## <a name="create-an-os-disk-with-a-tier-higher-than-the-baseline-tier-from-an-azure-marketplace-image"></a>从 Azure 市场映像创建一个层级高于基线层的 OS 磁盘

```azurecli
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
performanceTier=<yourDesiredPerformanceTier>
region=chinaeast
image=Canonical:UbuntuServer:18.04-LTS:18.04.202002180

az disk create -n $diskName -g $resourceGroupName -l $region --image-reference $image --sku Premium_LRS --tier $performanceTier
```

## <a name="update-the-tier-of-a-disk"></a>更新磁盘层级

```azurecli
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
performanceTier=<yourDesiredPerformanceTier>

az disk update -n $diskName -g $resourceGroupName --set tier=$performanceTier
```
## <a name="show-the-tier-of-a-disk"></a>显示磁盘层级

```azurecli
az disk show -n $diskName -g $resourceGroupName --query [tier] -o tsv
```

## <a name="next-steps"></a>后续步骤

如果需要调整磁盘大小以利用更高的性能层，请参阅以下文章：

- [使用 Azure CLI 扩展 Linux VM 上的虚拟硬盘](linux/expand-disks.md)
- [展开附加到 Windows 虚拟机的托管磁盘](https://ocs.microsoft.com/windows/expand-os-disk)

<!-- Update_Description: new article about disks performance tiers -->
<!--NEW.date: 11/02/2020-->