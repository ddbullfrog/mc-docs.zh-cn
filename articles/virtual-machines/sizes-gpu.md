---
title: Azure VM 大小 - GPU | Azure
description: 列出了 Azure 中虚拟机可用的不同 GPU 优化大小。 针对此系列中的大小列出了 vCPU、数据磁盘和 NIC 的数量，以及存储吞吐量和网络带宽。
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.workload: infrastructure-services
origin.date: 02/03/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 387589bc1969f6571248770c36a6b3ce44e56abf
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106244"
---
# <a name="gpu-optimized-virtual-machine-sizes"></a>GPU 优化虚拟机大小

GPU 优化 VM 大小是具有单个、多个或部分 GPU 的专用虚拟机。 这些大小是针对计算密集型、图形密集型和可视化工作负荷设计的。 本文介绍有关 GPU、vCPU、数据磁盘和 NIC 的数量和类型的信息。 此分组中的每个大小还包括存储吞吐量及网络带宽。

- [NCv3 系列](ncv3-series.md)大小已针对计算密集型 GPU 加速应用程序进行了优化。 一些示例包括基于 CUDA 和 OpenCL 的应用程序以及模拟、AI 和深度学习。 NC T4 v3 系列专用于采用 NVIDIA Tesla T4 GPU 和 AMD EPYC2 Rome 处理器的推理工作负荷。 NCv3 系列专用于采用 NVIDIA Tesla V100 GPU 的高性能计算和 AI 工作负荷。

    <!--Not Available on [NC T4_v3-series](nct4-v3-series.md)-->
    <!--Not Available on [NDv2-series](ndv2-series.md)-->
    <!--Not Available on [NV-series](nv-series.md)-->
    <!--Not Available on [NVv3-series](nvv3-series.md)-->
    <!--Not Available on [NVv4-series](nvv4-series.md)-->

## <a name="supported-operating-systems-and-drivers"></a>支持的操作系统和驱动程序

若要利用 Azure NCv3 系列 VM 的 GPU 功能，必须安装 NVIDIA 驱动程序。

<!--Not Available on  or AMD GPU which installed on NVV4 series-->
<!--CORRECT ON NCv3 series-->
<!--Not Available on [NVIDIA GPU Driver Extension](/virtual-machines/extensions/hpccompute-gpu-windows)-->

可以手动安装 NVIDIA GPU 驱动程序。 请参阅[在运行 Windows 的 N 系列 VM 上安装 NVIDIA GPU 驱动程序](./windows/n-series-driver-setup.md)或[在运行 Linux 的 N 系列 VM 上安装 NVIDIA GPU 驱动程序](./linux/n-series-driver-setup.md)，以了解受支持的操作系统、驱动程序以及安装和验证步骤。

<!--Not Available on [Install AMD GPU drivers on N-series VMs running Windows](./windows/n-series-amd-driver-setup.md)-->
<!--Which installed on NVV4 series-->

## <a name="deployment-considerations"></a>部署注意事项

- 有关 N 系列 VM 的可用性，请查看[可用产品(按区域)](https://azure.microsoft.com/regions/services/)。

- N 系列 VM 只能按 Resource Manager 部署模型部署。

- N 系列的 VM 在对其磁盘支持的 Azure 存储类型方面有所不同。 NCv3 VM 仅支持高级磁盘存储 (SSD) 所支持的 VM 磁盘。
    
    <!--ONLY AVAILABLE ON NCv3 series-->
    <!--Not Avaialble on NC and NV VMs only support VM disks that are backed by Standard Disk Storage (HDD).-->
    <!--Not Available on NCv2, ND, NDv2, and NVv2-->
    
- 如果需要部署的 N 系列 VM 较多，请考虑使用标准预付费套餐订阅或其他购买选项。 如果使用的是 [Azure 试用帐户](https://www.azure.cn/pricing/1rmb-trial/)，则仅可使用有限数量的 Azure 计算核心。

- 可能需要提高 Azure 订阅中的核心配额（按区域）以及单独针对 NCv3 核心的配额。 若要请求增加配额，可免费 [建立联机客户支持请求](https://support.azure.cn/support/support-azure/) 。 默认限制可能因订阅类别而异。
    
    <!--Not Available on NC, NCv2, ND, NDv2, NV, or NVv2-->
    <!--CORRECT ON https://support.azure.cn/support/support-azure/-->
    
## <a name="other-sizes"></a>其他大小

- [常规用途](sizes-general.md)
- [计算优化](sizes-compute.md)
    
    <!--Not Available on - [High performance compute](sizes-hpc.md)-->
    
- [内存优化](sizes-memory.md)

    <!--Not Available on - [Storage optimized](sizes-storage.md)-->
    
- [前几代](sizes-previous-gen.md)

## <a name="next-steps"></a>后续步骤

了解有关 [Azure 计算单元 (ACU)](acu.md) 如何帮助跨 Azure SKU 比较计算性能的详细信息。

<!-- Update_Description: update meta properties, wording update, update link -->