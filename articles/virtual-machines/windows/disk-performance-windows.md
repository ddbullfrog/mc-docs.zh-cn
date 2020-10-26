---
title: 虚拟机和磁盘性能
description: 详细了解如何组合使用 VM 及其附加的磁盘以提高性能
origin.date: 09/25/2020
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: no
ms.testdate: 10/19/2020
ms.author: v-yeche
ms.topic: conceptual
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: 101f69fdf74caa1ffbb596b2ecb338e6aa799714
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128856"
---
<!--Notice: Two Includes File in the new file-->
# <a name="virtual-machine-and-disk-performance"></a>虚拟机和磁盘性能
[!INCLUDE [VM and Disk Performance](../../../includes/virtual-machine-disk-performance.md)]

## <a name="virtual-machine-uncached-vs-cached-limits"></a>虚拟机非缓存限制与缓存限制
 同时启用了高级存储和高级存储缓存的虚拟机有两种不同的存储带宽限制。 接下来，让我们继续查看 Standard_D8s_v3 虚拟机的示例。 下面是有关 [Dsv3 系列](../dv3-dsv3-series.md)及其中的 Standard_D8s_v3 的文档：

[!INCLUDE [VM and Disk Performance](../../../includes/virtual-machine-disk-performance-2.md)]

<!-- Update_Description: new article about disk performance windows -->
<!--NEW.date: 10/19/2020-->