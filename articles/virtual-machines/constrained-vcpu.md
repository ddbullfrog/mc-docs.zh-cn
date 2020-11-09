---
title: 受约束的 vCPU 大小
description: 列出支持具有受约束的 vCPU 计数的 VM 大小。
ms.service: virtual-machines
ms.topic: conceptual
origin.date: 03/09/2018
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 11/02/2020
ms.author: v-yeche
ms.openlocfilehash: 64185a6fe8a5f2de55a5436cec8b2d5bbdacd60b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105126"
---
# <a name="constrained-vcpu-capable-vm-sizes"></a>支持受约束 vCPU 的 VM 大小

某些数据库工作负荷（如 SQL Server 或 Oracle）需要高内存、存储和 I/O 带宽，但不需要高核心计数。 许多数据库工作负荷不是 CPU 密集型工作负荷。 Azure 提供了某些 VM 大小（其中你可以限制 VM vCPU 计数），以降低软件许可成本，同时保持相同的内存、存储和 I/O 带宽。

可将 vCPU 计数限制为一半或四分之一的原始 VM 大小。 这些新 VM 大小具有可指定活动的 vCPU 数的后缀，以便可以更轻松地识别。

<!--Not Avaialble on GS series-->
<!--Not Avaialble on For example, the current VM size Standard_GS5 comes with 32 vCPUs, 448 GB RAM, 64 disks (up to 256 TB), and 80,000 IOPs or 2 GB/s of I/O bandwidth. The new VM sizes Standard_GS5-16 and Standard_GS5-8 comes with 16 and 8 active vCPUs respectively, while maintaining the rest of the specs of the Standard_GS5 for memory, storage, and I/O bandwidth.-->


为 SQL Server 或 Oracle 收取的许可费受限于新的 vCPU 计数，对其他产品应基于新的 vCPU 计数收取费用。 这会导致 VM 规格与活动（可计费）vCPU 数的比率增加 50% 到 75%。 这些新的 VM 大小允许客户工作负荷使用相同的内存、存储和 I/O 带宽，同时优化其软件许可成本。 目前，计算成本（包括 OS 许可）与原始大小保持相同成本。 有关详细信息，请参阅 [Azure VM sizes for more cost-effective database workloads](https://azure.microsoft.com/blog/announcing-new-azure-vm-sizes-for-more-cost-effective-database-workloads/)（适用于更经济高效数据库工作负荷的 Azure VM 大小）。

| 名称                | vCPU | 规格           |
|---------------------|------|-----------------|
| Standard_M8-2ms     | 2    | 与 M8ms 相同    |
| Standard_M8-4ms     | 4    | 与 M8ms 相同    |
| Standard_M16-4ms    | 4    | 与 M16ms 相同   |
| Standard_M16-8ms    | 8    | 与 M16ms 相同   |
| Standard_M32-8ms    | 8    | 与 M32ms 相同   |
| Standard_M32-16ms   | 16   | 与 M32ms 相同   |
| Standard_M64-32ms   | 32   | 与 M64ms 相同   |
| Standard_M64-16ms   | 16   | 与 M64ms 相同   |
| Standard_M128-64ms  | 64   | 与 M128ms 相同  |
| Standard_M128-32ms  | 32   | 与 M128ms 相同  |
| Standard_E4-2s_v3   | 2    | 与 E4s_v3 相同  |
| Standard_E8-4s_v3   | 4    | 与 E8s_v3 相同  |
| Standard_E8-2s_v3   | 2    | 与 E8s_v3 相同  |
| Standard_E16-8s_v3  | 8    | 与 E16s_v3 相同 |
| Standard_E16-4s_v3  | 4    | 与 E16s_v3 相同 |
| Standard_E32-16s_v3 | 16   | 与 E32s_v3 相同 |
| Standard_E32-8s_v3  | 8    | 与 E32s_v3 相同 |
| Standard_E64-32s_v3 | 32   | 与 E64s_v3 相同 |
| Standard_E64-16s_v3 | 16   | 与 E64s_v3 相同 |
| Standard_E4-2s_v4   | 2    | 与 E4s_v4 相同  |
| Standard_E8-4s_v4   | 4    | 与 E8s_v4 相同  |
| Standard_E8-2s_v4   | 2    | 与 E8s_v4 相同  |
| Standard_E16-8s_v4  | 8    | 与 E16s_v4 相同 |
| Standard_E16-4s_v4  | 4    | 与 E16s_v4 相同 |
| Standard_E32-16s_v4 | 16   | 与 E32s_v4 相同 |
| Standard_E32-8s_v4  | 8    | 与 E32s_v4 相同 |
| Standard_E64-32s_v4 | 32   | 与 E64s_v4 相同 |
| Standard_E64-16s_v4 | 16   | 与 E64s_v4 相同 |
| Standard_E4-2ds_v4  | 2    | 与 E4ds_v4 相同 |
| Standard_E8-4ds_v4  | 4    | 与 E8ds_v4 相同 |
| Standard_E8-2ds_v4  | 2    | 与 E8ds_v4 相同 |
| Standard_E16-8ds_v4 | 8    | 与 E16ds_v4 相同|
| Standard_E16-4ds_v4 | 4    | 与 E16ds_v4 相同|
| Standard_E32-16ds_v4| 16   | 与 E32ds_v4 相同|
| Standard_E32-8ds_v4 | 8    | 与 E32ds_v4 相同|
| Standard_E64-32ds_v4| 32   | 与 E64ds_v4 相同|
| Standard_E64-16ds_v4| 16   | 与 E64ds_v4 相同|
| Standard_GS4-8      | 8    | 与 GS4 相同     |
| Standard_GS4-4      | 4    | 与 GS4 相同     |
| Standard_GS5-16     | 16   | 与 GS5 相同     |
| Standard_GS5-8      | 8    | 与 GS5 相同     |
| Standard_DS11-1_v2  | 1    | 与 DS11_v2 相同 |
| Standard_DS12-2_v2  | 2    | 与 DS12_v2 相同 |
| Standard_DS12-1_v2  | 1    | 与 DS12_v2 相同 |
| Standard_DS13-4_v2  | 4    | 与 DS13_v2 相同 |
| Standard_DS13-2_v2  | 2    | 与 DS13_v2 相同 |
| Standard_DS14-8_v2  | 8    | 与 DS14_v2 相同 |
| Standard_DS14-4_v2  | 4    | 与 DS14_v2 相同 |

<!--Not Available on Standard_GS-->
<!--Not Available on Standard_M416-208s_v2-->

## <a name="other-sizes"></a>其他大小
- [计算优化](./sizes-compute.md)
- [内存优化](./sizes-memory.md)

    <!--Not Available on - [Storage optimized](./sizes-storage.md)-->
    
- [GPU](./sizes-gpu.md)

<!--Not Available on - [High performance compute](./sizes-hpc.md)-->

## <a name="next-steps"></a>后续步骤
了解有关 [Azure 计算单元 (ACU)](./acu.md) 如何帮助跨 Azure SKU 比较计算性能的详细信息。

<!-- Update_Description: new article about constrained vcpu -->
<!--NEW.date: 09/07/2020-->