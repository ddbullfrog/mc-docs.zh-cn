---
title: VM 大小
description: 列出 Azure 中虚拟机的不同可用大小。
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.workload: infrastructure-services
origin.date: 07/21/2020
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: yes
ms.testdate: 10/19/2020
ms.author: v-yeche
ms.openlocfilehash: f3f7c787265a18bae473508e756b93b99bad0125
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128097"
---
<!--Verified Successfully-->
# <a name="sizes-for-virtual-machines-in-azure"></a>Azure 中虚拟机的大小

本文介绍可用于运行应用和工作负载的 Azure 虚拟机的可用大小与选项。 此外，还提供在计划使用这些资源时要考虑的部署注意事项。 

| 类型 | 大小 | 说明 |
|------|-------|-------------|
| [常规用途](sizes-general.md)   | B、Dsv3、Dv3、DSv2、Dv2、Av2、Dv4、Dsv4、Ddv4、Ddsv4 | CPU 与内存之比平衡。 适用于测试和开发、小到中型数据库和低到中等流量 Web 服务器。 |
| [计算优化](sizes-compute.md) | F、Fs、Fsv2 | 高 CPU 与内存之比。 适用于中等流量的 Web 服务器、网络设备、批处理和应用程序服务器。 |
| [内存优化](sizes-memory.md) | Esv3、Ev3、Ev4、Esv4、Edv4、Edsv4、M、DSv2、Dv2  | 高内存与 CPU 之比。 适用于关系数据库服务器、中到大型规模的缓存和内存中分析。 |

<!-- Not Available  Dasv4, Dav4, DC, DCv2 -->
<!-- Not Available  Easv4, Eav4, Mv2, -->
<!-- Not Available Storage optimized        | Lsv2 -->
<!-- Not Available GPU            | NC, NCv2, NCv3, NCasT4_v3 (Preview), ND, NDv2 (Preview), NV, NVv3, NVv4  -->
<!-- Not Available High performance compute | HB, HBv2, HC, H-->

- 有关不同大小的定价信息，请参阅 [Linux](https://www.azure.cn/pricing/details/virtual-machines/) 或 [Windows](https://www.azure.cn/pricing/details/virtual-machines/#Windows) 的定价页。
- 如需了解 Azure 区域中各种 VM 大小的可用性，请参阅 [可用产品（按区域）](https://azure.microsoft.com/regions/services/)。
- 若要查看 Azure VM 的一般限制，请参阅 [Azure 订阅和服务限制、配额与约束](../azure-resource-manager/management/azure-subscription-service-limits.md)。
- 有关 Azure 如何命名其 VM 的详细信息，请参阅 [Azure 虚拟机大小命名约定](./vm-naming-conventions.md)。

## <a name="rest-api"></a>REST API

有关使用 REST API 来查询 VM 大小的信息，请参阅以下文章：

- [List available virtual machine sizes for resizing](https://docs.microsoft.com/rest/api/compute/virtualmachines/listavailablesizes)（列出可用的虚拟机大小以便调整大小）
- [List available virtual machine sizes for a subscription](https://docs.microsoft.com/rest/api/compute/resourceskus/list)（列出订阅的可用虚拟机大小）
- [List available virtual machine sizes in an availability set](https://docs.microsoft.com/rest/api/compute/availabilitysets/listavailablesizes)（列出可用性集中的可用虚拟机大小）

## <a name="acu"></a>ACU

了解有关 [Azure 计算单元 (ACU)](acu.md) 如何帮助跨 Azure SKU 比较计算性能的详细信息。

## <a name="benchmark-scores"></a>基准评分

使用 [CoreMark 基准测试分数](./linux/compute-benchmark-scores.md)，详细了解 Linux VM 的计算性能。

使用 [SPECInt 基准测试分数](./windows/compute-benchmark-scores.md)，详细了解 Windows VM 的计算性能。

<!--Not Available on ## Manage costs-->

<!--Not Suitable on [!INCLUDE [cost-management-horizontal](../../includes/cost-management-horizontal.md)-->
<!--For cost-management-billing is not Available on Azure China-->

## <a name="next-steps"></a>后续步骤

了解关于可用的各种 VM 大小的详细信息：

- [常规用途](sizes-general.md)
- [计算优化](sizes-compute.md)
- [内存优化](sizes-memory.md)
    
    <!--Not Avaialble on - [Storage optimized](sizes-storage.md)-->
    
- [GPU](sizes-gpu.md)
    
    <!-- Not Available on - [High performance compute](sizes-hpc.md)-->
    
- 查看[上一代](sizes-previous-gen.md)页面，了解 A Standard、Dv1（D1-4 和 D11-14 v1）以及 A8-A11 系列

<!-- Update_Description: update meta properties, wording update, update link -->