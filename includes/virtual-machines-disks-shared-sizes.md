---
title: include 文件
description: include 文件
services: virtual-machines
ms.service: virtual-machines
ms.topic: include
origin.date: 04/06/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: yes|no
ms.testdate: 11/09/2020null
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: cf8a49dc4e9e36892079c9d6e50d30ec212ed535
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328993"
---
<!--Verified successfully for PG notification-->
目前，只有高级 SSD 可启用共享磁盘。 如果磁盘大小不同，则 `maxShares` 限制可能也不同，设置 `maxShares` 值时不可超过此限制。 对于高级 SSD，支持共享磁盘的磁盘大小至少是 P15。

<!--Not Available on FEATURE ultra disks-->

对于每个磁盘，可定义一个 `maxShares` 值来表示可同时共享磁盘的最大节点数。 例如，如果计划设置一个双节点故障转移群集，可设置 `maxShares=2`。 上限就是最大值。 只要节点数低于指定的 `maxShares` 值，节点就可加入或离开群集（装载或卸载磁盘）。

> [!NOTE]
> 仅在磁盘从所有节点中拆离后，还可设置或编辑 `maxShares` 值。

### <a name="premium-ssd-ranges"></a>高级·SSD 范围

下表说明了按高级磁盘大小得出的 `maxShares` 的最大允许值：

|磁盘大小  |maxShares 限制  |
|---------|---------|
|P15、P20     |2         |
|P30、P40、P50     |5         |
|P60、P70、P80     |10         |

磁盘的 IOPS 和带宽限制不受 `maxShares` 值影响。 例如，无论 maxShares = 1 还是 maxShares > 1，P15 磁盘的 IOPS 上限都为 1100。

<!--Not Available on ### Ultra disk ranges-->

<!-- Update_Description: new article about virtual machines disks shared sizes -->
<!--NEW.date: 11/09/2020-->