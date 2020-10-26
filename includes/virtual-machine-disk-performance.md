---
title: include 文件
description: include 文件
services: virtual-machines
ms.service: virtual-machines
ms.topic: include
origin.date: 07/07/2020
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: yes|no
ms.testdate: 10/19/2020null
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: 6b9afc743459f29a0b7d1eb6f2536f1a2a0bc62f
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128859"
---
本文介绍了磁盘性能，以及在将 Azure 虚拟机和 Azure 磁盘组合使用时磁盘性能的工作原理。 此外还介绍了如何诊断磁盘 IO 的瓶颈，以及可以进行哪些更改以优化性能。

## <a name="how-does-disk-performance-work"></a>磁盘性能工作原理
Azure 虚拟机具有 IOPS 和吞吐量性能限制，这些限制由虚拟机类型和大小决定。 可附加到虚拟机的 OS 磁盘和数据磁盘具有自己的 IOPS 和吞吐量限制。 虚拟机上运行的应用程序所请求的 IOPS 或吞吐量大于为虚拟机或附加磁盘分配的 IOPS 或吞吐量时，应用程序的性能将受到限制。 发生这种情况时，应用程序将性能欠佳，并可能导致延迟时间增加等负面后果。 让我们浏览几个示例来巩固这些知识。 为了使这些示例易于理解，我们只讨论 IOPS，但同样的逻辑也适用于吞吐量。

## <a name="disk-io-capping"></a>磁盘 IO 上限
设置：
- Standard_D8s_v3 
    - 未缓存的 IOPS：12,800
- E30 OS 磁盘
    - IOPS：500 
- 2 个 E30 数据磁盘
    - IOPS：500

![磁盘级别上限](media/vm-disk-performance/disk-level-throttling.jpg)

在虚拟机上运行的应用程序向虚拟机发出要求 10,000 个 IOPS 的请求。 VM 允许所有这些请求，因为 Standard_D8s_v3 虚拟机最多可以执行 12,800 个 IOPS。 这些要求 10,000 个 IOPS 的请求随后会被分解为对不同磁盘的三个不同请求。 向操作系统磁盘请求 1,000 个 IOPS，向每个数据磁盘请求 4,500 个 IOPS。 由于所有附加的磁盘都是 E30 磁盘，只能处理 500 个 IOPS，因此这些磁盘每个都以 500 个 IOPS 的速度响应。 应用程序的性能因此会受到附加磁盘的限制，只能处理 1,500 个 IOPS。 如果使用性能更好的磁盘（例如，高级 SSD P30 磁盘），则峰值性能可以达到 10,000 个 IOPS。

## <a name="virtual-machine-io-capping"></a>虚拟机 IO 上限
设置：
- Standard_D8s_v3 
    - 未缓存的 IOPS：12,800
- P30 OS 磁盘
    - IOPS：5,000 
- 2 个 P30 数据磁盘 
    - IOPS：5,000

![虚拟机级别上限](media/vm-disk-performance/vm-level-throttling.jpg)

在虚拟机上运行的应用程序发出要求 15,000 个 IOPS 的请求。 遗憾的是，Standard_D8s_v3 虚拟机仅预配为处理 12,800 个 IOPS。 因此，应用程序需遵循虚拟机限制，必须对分配给它的 12,800 个 IOPS 进行分配。 这些要求 12,800 个 IOPS 的请求随后会被分解为对不同磁盘的三个不同请求。 向操作系统磁盘请求 4,267 个 IOPS，向每个数据磁盘请求 4,266 个 IOPS。 由于附加的所有磁盘都是 P30 磁盘（可以处理 5,000 个 IOPS），因此它们会以请求的数量进行响应。

<!-- Update_Description: new article about virtual machine disk performance -->
<!--NEW.date: 10/19/2020-->