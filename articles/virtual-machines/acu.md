---
title: Azure 计算单位概述
description: Azure 计算单位的概念概述。 ACU 提供了一种在 Azure SKU 中比较 CPU 性能的方法。
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.workload: infrastructure-services
origin.date: 02/03/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: no
ms.testdate: 10/26/2020
ms.author: v-yeche
ms.reviewer: davberg
ms.openlocfilehash: eb92b5219102e46534fec908f835df13adc23645
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106253"
---
<!--Verified successfully-->
# <a name="azure-compute-unit-acu"></a>Azure 计算单元 (ACU)

Azure 计算单位 (ACU) 这一概念提供一种比较 Azure SKU 的计算 (CPU) 性能的方法。 这有助于轻松确定最有可能满足性能需求的 SKU。 ACU 目前在数量为 100 的小型 (Standard_A1) VM 上标准化，而所有其他 SKU 则表示该 SKU 在运行标准基准测试时大约会快多少

*ACU 使用 Intel® Turbo 技术来增加 CPU 频率和提升性能。  性能提升程度可能因 VM 大小、工作负荷和同一主机上运行的其他工作负荷而有所不同。

**ACU 使用 AMD® Boost 技术来增加 CPU 频率和提升性能。  性能提升程度可能因 VM 大小、工作负荷和同一主机上运行的其他工作负荷而有所不同。

**_超线程，能够运行嵌套虚拟化

> [!IMPORTANT]
> ACU 只是一种规则。 工作负荷的结果可能会有所不同。
<br />

| SKU 系列 | ACU\vCPU | vCPU：核心 |
| --- | --- |---|
| [A0](sizes-previous-gen.md) |50 | 1:1 |
| [A1 - A4](sizes-previous-gen.md) |100 | 1:1 |
| [A5 - A7](sizes-previous-gen.md) |100 | 1:1 |
| [A1_v2 - A8_v2](sizes-general.md) |100 | 1:1 |
| [A2m_v2 - A8m_v2](sizes-general.md) |100 | 1:1 |
| [B](sizes-b-series-burstable.md) |多种多样 | 多种多样 |
| [D1 - D14](sizes-previous-gen.md) |160 - 250 | 1:1 |
| [D1_v2 - D15_v2](dv2-dsv2-series.md) |210 - 250_ | 1:1 |
| [DS1 - DS14](sizes-previous-gen.md) |160 - 250 | 1:1 |
| [DS1_v2 - DS15_v2](dv2-dsv2-series.md) |210 - 250* | 1:1 |
| [D_v3](dv3-dsv3-series.md) |160 - 190* | 2:1\*\*\* |
| [Ds_v3](dv3-dsv3-series.md) |160 - 190* | 2:1\*\*\* |
| [Dv4](dv4-dsv4-series.md) | 195 - 210 | 2:1\*\*\* |
| [Dsv4](dv4-dsv4-series.md) | 195 - 210 | 2:1\*\*\* |
| [Ddv4](ddv4-ddsv4-series.md) | 195 -210* | 2:1\*\*\* |
| [Ddsv4](ddv4-ddsv4-series.md) | 195 - 210* | 2:1\*\*\* |
| [E_v3](ev3-esv3-series.md) |160 - 190* | 2:1\*\*\*|
| [Es_v3](ev3-esv3-series.md) |160 - 190* | 2:1\*\*\* |
| [Ev4](ev4-esv4-series.md) | 195 - 210 | 2:1\*\*\* |
| [Esv4](ev4-esv4-series.md) | 195 - 210 | 2:1\*\*\* |
| [Edv4](edv4-edsv4-series.md) | 195 - 210* | 2:1\*\*\* |
| [Edsv4](edv4-edsv4-series.md) | 195 - 210* | 2:1\*\*\* |
| [F2s_v2 - F72s_v2](fsv2-series.md) |195 - 210* | 2:1\*\*\* |
| [F1 - F16](sizes-previous-gen.md) |210 - 250* | 1:1 |
| [F1s - F16s](sizes-previous-gen.md) |210 - 250* | 1:1 |
| [M](m-series.md) | 160 - 180 | 2:1\*\*\* |

<!-- Not Available  [A8-A11]  -->
<!-- Not Available  [Da*v4]  -->
<!-- Not Available  [Ea*v4]  -->
<!-- Not Available  [G1-G5]  -->
<!-- Not Available  [GS1-GS5]  -->
<!-- Not Available  [H] -->
<!-- Not Available  [HB] -->
<!-- Not Available  [HC] -->
<!-- Not Available  [L4s - L32s]  -->
<!-- Not Available  [L8s_v2 - L80s_v2]  -->
<!-- Not Available  [NVv4]  -->

有关各种大小的详细信息，请访问以下链接：

- [通用](sizes-general.md)
- [内存优化](sizes-memory.md)
- [计算优化](sizes-compute.md)
- [GPU 优化](sizes-gpu.md)

<!--Not Available on - [High performance compute](sizes-hpc.md)-->
<!--Not Available on - [Storage optimized](sizes-storage.md)-->
<!-- Update_Description: update meta properties, wording update, update link -->