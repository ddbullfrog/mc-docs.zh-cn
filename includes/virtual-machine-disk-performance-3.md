---
title: include 文件
description: include 文件
services: virtual-machines
ms.service: virtual-machines
ms.topic: include
origin.date: 10/12/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: no
ms.testdate: 11/02/2020
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: 84afafcc16eaee62511d0fa17b30b807bb436a92
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104875"
---
<!--Verified Successfully-->
![指标菜单](media/vm-disk-performance/utilization-metrics-example/fio-output.jpg)

但是，Standard_D8s_v3 可实现总计 28,600 的 IOPS，使用指标可以调查正在进行的操作并识别存储 IO 瓶颈。 首先，找到指标按钮左侧菜单并选择它：

![指标菜单](media/vm-disk-performance/utilization-metrics-example/metrics-menu.jpg)

我们先看一下 **已使用的 VM 缓存 IOPS 的百分比** 指标：

![已使用的 VM 缓存 IOPS 的百分比](media/vm-disk-performance/utilization-metrics-example/vm-cached.jpg)

此指标告诉我们，在 VM 上分配给缓存 IOPS 的 16,000 IOPS 中，使用了 61%。 这就意味着存储 IO 瓶颈与缓存的磁盘无关，因为该指标值没有达到 100%。 所以我们现在来看一下 **已使用的 VM 未缓存 IOPS 的百分比** 指标：

![已使用的 VM 未缓存 IOPS 的百分比](media/vm-disk-performance/utilization-metrics-example/vm-uncached.jpg)

此指标值已达到 100%，这告诉我们，在 VM 上分配给未缓存 IOPS 的所有 12,800 IOPS 均已使用。 一种可以修正这种情况的方法是，将 VM 大小更改为可处理更多 IO 的更大的大小。 但在执行此操作之前，我们先来看一下附加的磁盘，以了解这些磁盘发生了多少 IOPS。 我们先通过查看 **已使用的 OS 磁盘 IOPS 的百分比** 来看一下 OS 磁盘：

![已使用的 OS 磁盘 IOPS 的百分比](media/vm-disk-performance/utilization-metrics-example/os-disk.jpg)

此指标告诉我们，在为此 P30 OS 磁盘预配的 5,000 IOPS 中，使用了大约 90%。 这就意味着在 OS 磁盘上没有瓶颈。 现在我们通过查看 **已使用的数据磁盘 IOPS 的百分比** 来看一下附加到 VM 的数据磁盘：

![已使用的数据磁盘 IOPS 的百分比](media/vm-disk-performance/utilization-metrics-example/data-disks-no-splitting.jpg)

此指标告诉我们，在所有附加的磁盘上，平均已使用的 IOPS 百分比约为 42%。 此百分比是根据由这些磁盘使用且未从主机缓存中予以服务的 IOPS 计算得出的。 让我们通过对这些指标应用拆分并按 LUN 值拆分来深入了解此指标：

![使用拆分的已使用的数据磁盘 IOPS 的百分比](media/vm-disk-performance/utilization-metrics-example/data-disks-splitting.jpg)

此指标告诉我们，LUN 3 和 2 上附加的数据磁盘使用了它们约 85% 的预配 IOPS。 下面是 VM 和磁盘体系结构中 IO 情况的示意图：

![存储 IO 指标示例图](media/vm-disk-performance/utilization-metrics-example/metrics-diagram.jpg)

<!-- Update_Description: new article about virtual machine disk performance 3 -->
<!--NEW.date: 11/02/2020-->