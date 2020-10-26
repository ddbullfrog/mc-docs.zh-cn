---
title: Azure VM 大小 - 前几代 | Azure
description: 列出 Azure 中可用的前几代虚拟机大小。 针对此系列中的大小列出了 vCPU、数据磁盘和 NIC 的数量，以及存储吞吐量和网络带宽。
services: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.workload: infrastructure-services
origin.date: 02/20/2020
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: no
ms.testdate: 10/19/2020
ms.author: v-yeche
ms.openlocfilehash: 0bc96a9022665845c5800500267311b18455a373
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92127917"
---
# <a name="previous-generations-of-virtual-machine-sizes"></a>前几代虚拟机大小

本部分提供了有关先前几代虚拟机大小的信息。 这些大小仍可使用，但有新的大小可供使用。

## <a name="f-series"></a>F 系列

F 系列基于 2.4 GHz Intel Xeon® E5-2673 v3 (Haswell) 处理器，该处理器使用 Intel Turbo Boost 技术 2.0，可实现高达 3.1 GHz 的时钟速度。 此 CPU 性能与 Dv2 系列的 VM 相同。  

对于需要更快的 CPU，但是每个 vCPU 不需要太多内存或临时存储的工作负荷来说，F 系列 VM 是一个极佳选择。  诸如分析、游戏服务器、Web 服务器和批处理等工作负荷从 F 系列的优势中获益。

ACU：210 - 250

高级存储：不支持

高级存储缓存：不支持

<!--MOONCAKE: CORRECT ON Max NICs / Expected network bandwidth (Mbps)-->

| 大小 | vCPU | 内存:GiB | 临时存储 (SSD) GiB | 最大临时存储吞吐量：IOPS/读取 MBps/写入 MBps | 最大数据磁盘数/吞吐量：IOPS | 最大 NIC 数/预期网络带宽 (Mbps) |
|---|---|---|---|---|---|---|
| Standard_F1  | 1  | 2  | 16  | 3000/46/23    | 4/4x500   | 2/750   |
| Standard_F2  | 2  | 4  | 32  | 6000/93/46    | 8/8x500   | 2/1500  |
| Standard_F4  | 4  | 8  | 64  | 12000/187/93  | 16/16x500 | 4/3000  |
| Standard_F8  | 8  | 16 | 128 | 24000/375/187 | 32/32x500 | 8/6000  |
| Standard_F16 | 16 | 32 | 256 | 48000/750/375 | 64/64x500 | 8/12000 |

## <a name="fs-series-sup1sup"></a>Fs 系列 <sup>1</sup>

Fs 系列具有 F 系列的所有优势（在高级存储的基础上）。

ACU：210 - 250

高级存储：支持

高级存储缓存：支持

<!--MOONCAKE: CORRECT ON Max NICs / Expected network bandwidth (Mbps)-->

| 大小 | vCPU | 内存:GiB | 临时存储 (SSD) GiB | 最大数据磁盘数 | 最大缓存吞吐量和临时存储吞吐量：IOPS/MBps（以 GiB 为单位的缓存大小） | 最大非缓存磁盘吞吐量：IOPS/MBps | 最大 NIC 数/预期网络带宽 (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_F1s  | 1  | 2  | 4  | 4  | 4000/32 (12)    | 3200/48   | 2/750   |
| Standard_F2s  | 2  | 4  | 8  | 8  | 8000/64 (24)    | 6400/96   | 2/1500  |
| Standard_F4s  | 4  | 8  | 16 | 16 | 16000/128 (48)  | 12800/192 | 4/3000  |
| Standard_F8s  | 8  | 16 | 32 | 32 | 32000/256 (96)  | 25600/384 | 8/6000  |
| Standard_F16s | 16 | 32 | 64 | 64 | 64000/512 (192) | 51200/768 | 8/12000 |

Mbps = 每秒 10^6 字节，GiB = 1024^3 字节。

<sup>1</sup> Fs 系列 VM 可能的最大磁盘吞吐量（IOPS 或 Mbps）可能受限于附加磁盘的数量、大小和条带化。  有关详细信息，请参阅[高性能设计](premium-storage-performance.md)。

<!--Not Available on ## NVv2-series-->

## <a name="older-generations-of-virtual-machine-sizes"></a>前几代虚拟机大小

本部分提供了有关先前几代虚拟机大小的信息。 这些大小仍受支持，但无法接收更多的容量。 有更新的或备用的大小，这些大小已公开发布。 请参阅 [Azure 中 Linux 虚拟机的大小](./sizes.md)，选择最适合自己需求的 VM 大小。  

有关如何调整 Linux VM 的大小的详细信息，请参阅[调整 Linux VM 的大小](linux/change-vm-size.md)。  

<br />

### <a name="basic-a"></a>基本 A  

**更新的大小建议** ： [Av2 系列](av2-series.md)

高级存储：不支持

高级存储缓存：不支持

基本层大小主要用于开发工作负荷，以及其他不需要负载均衡、自动缩放或内存密集型虚拟机的应用程序。

| 大小 - 大小\名称 | vCPU | 内存|NIC 数（最大值）| 最大临时磁盘大小 | 最大 数据磁盘（每个 1023 GB）| 最大 IOPS（每个磁盘 300 次） |
|---|---|---|---|---|---|---|
| A0\Basic_A0 | 1 | 768 MB  | 2 | 20 GB  | 1  | 1x300  |
| A1\Basic_A1 | 1 | 1.75 GB | 2 | 40 GB  | 2  | 2x300  |
| A2\Basic_A2 | 2 | 3.5 GB  | 2 | 60 GB  | 4  | 4x300  |
| A3\Basic_A3 | 4 | 7 GB    | 2 | 120 GB | 8  | 8x300  |
| A4\Basic_A4 | 8 | 14 GB   | 2 | 240 GB | 16 | 16x300 |

<br />

### <a name="standard-a0---a4-using-cli-and-powershell"></a>使用 CLI 和 PowerShell 的标准 A0 - A4

在经典部署模型中，CLI 和 PowerShell 中的一些 VM 大小名称略有不同：

* Standard_A0 是特小型
* Standard_A1 是小型
* Standard_A2 是中型
* Standard_A3 是大型
* Standard_A4 是超大型

### <a name="a-series"></a>A 系列  

**更新的大小建议** ： [Av2 系列](av2-series.md)

ACU：50-100

高级存储：不支持

高级存储缓存：不支持

| 大小 | vCPU | 内存:GiB | 临时存储 (HDD)：GiB | 最大数据磁盘数 | 数据磁盘最大吞吐量：IOPS | 最大 NIC 数/预期网络带宽 (Mbps) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_A0&nbsp;<sup>1</sup> | 1 | 0.768 | 20 个 | 1 | 1x500 | 2/100 |
| Standard_A1 | 1 | 1.75 | 70  | 2  | 2x500  | 2/500  |
| Standard_A2 | 2 | 3.5  | 135 | 4  | 4x500  | 2/500  |
| Standard_A3 | 4 | 7    | 285 | 8  | 8x500  | 2/1000 |
| Standard_A4 | 8 | 14   | 605 | 16 | 16x500 | 4/2000 |
| Standard_A5 | 2 | 14   | 135 | 4  | 4x500  | 2/500  |
| Standard_A6 | 4 | 28   | 285 | 8  | 8x500  | 2/1000 |
| Standard_A7 | 8 | 56   | 605 | 16 | 16x500 | 4/2000 |

<sup>1</sup> A0 大小在物理硬件上过度订阅。 仅针对此特定大小，其他客户部署可能影响正在运行的工作负荷的性能。 以下概述的相对性能为预期的基准，受限于近似变化性的 15%。

<br />

<!--Not Available on ### A-series - compute-intensive instances-->

### <a name="d-series"></a>D 系列  

**更新的大小建议** ：[Dv4 系列](dv4-dsv4-series.md)和 [Ddv4 系列](ddv4-ddsv4-series.md)

<!--Not Available on [Dav4-series](dav4-dasv4-series.md)-->

ACU：160-250 <sup>1</sup>

高级存储：不支持

高级存储缓存：不支持

| 大小 | vCPU | 内存:GiB | 临时存储 (SSD) GiB | 最大临时存储吞吐量：IOPS/读取 MBps/写入 MBps | 最大数据磁盘数/吞吐量：IOPS | 最大 NIC 数/预期网络带宽 (Mbps) |
|---|---|---|---|---|---|---|
| Standard_D1  | 1 | 3.5 | 50  | 3000/46/23    | 4/4x500   | 2/500  |
| Standard_D2  | 2 | 7   | 100 | 6000/93/46    | 8/8x500   | 2/1000 |
| Standard_D3  | 4 | 14  | 200 | 12000/187/93  | 16/16x500 | 4/2000 |
| Standard_D4  | 8 | 28  | 400 | 24000/375/187 | 32/32x500 | 8/4000 |

<sup>1</sup> VM 系列可以在下述 CPU 之一上运行：2.2 GHz Intel Xeon® E5-2660 v2、2.4 GHz Intel Xeon® E5-2673 v3 (Haswell) 或 2.3 GHz Intel XEON® E5-2673 v4 (Broadwell)  

<br />

### <a name="d-series---memory-optimized"></a>D 系列 - 内存优化  

**更新的大小建议** ：[Dv4 系列](dv4-dsv4-series.md)和 [Ddv4 系列](ddv4-ddsv4-series.md)

<!--Not Available on [Dav4-series](dav4-dasv4-series.md)-->

ACU：160-250 <sup>1</sup>

高级存储：不支持

高级存储缓存：不支持

| 大小 | vCPU | 内存:GiB | 临时存储 (SSD) GiB | 最大临时存储吞吐量：IOPS/读取 MBps/写入 MBps | 最大数据磁盘数/吞吐量：IOPS | 最大 NIC 数/预期网络带宽 (Mbps) |
|---|---|---|---|---|---|---|
| Standard_D11 | 2  | 14  | 100 | 6000/93/46    | 8/8x500   | 2/1000 |
| Standard_D12 | 4  | 28  | 200 | 12000/187/93  | 16/16x500 | 4/2000 |
| Standard_D13 | 8  | 56  | 400 | 24000/375/187 | 32/32x500 | 8/4000 |
| Standard_D14 | 16 | 112 | 800 | 48000/750/375 | 64/64x500 | 8/8000 |

<sup>1</sup> VM 系列可以在下述 CPU 之一上运行：2.2 GHz Intel Xeon® E5-2660 v2、2.4 GHz Intel Xeon® E5-2673 v3 (Haswell) 或 2.3 GHz Intel XEON® E5-2673 v4 (Broadwell)  

<br />

<!--Not Available on ## Preview: DC-series-->

### <a name="ds-series"></a>DS 系列  

**更新的大小建议** ：[Dsv4 系列](dv4-dsv4-series.md)和 [Ddsv4 系列](ddv4-ddsv4-series.md)

<!--Not Available on [Dav4-series](dav4-dasv4-series.md)-->

ACU：160-250 <sup>1</sup>

高级存储：支持

高级存储缓存：支持

| 大小 | vCPU | 内存:GiB | 临时存储 (SSD) GiB | 最大数据磁盘数 | 最大缓存吞吐量和临时存储吞吐量：IOPS/MBps（以 GiB 为单位的缓存大小） | 最大非缓存磁盘吞吐量：IOPS/MBps | 最大 NIC 数/预期网络带宽 (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_DS1 | 1 | 3.5 | 7  | 4  | 4000/32 (43)    | 3200/32   | 2/500  |
| Standard_DS2 | 2 | 7   | 14 | 8  | 8000/64 (86)    | 6400/64   | 2/1000 |
| Standard_DS3 | 4 | 14  | 28 | 16 | 16000/128 (172) | 12800/128 | 4/2000 |
| Standard_DS4 | 8 | 28  | 56 | 32 | 32000/256 (344) | 25600/256 | 8/4000 |

<sup>1</sup> VM 系列可以在下述 CPU 之一上运行：2.2 GHz Intel Xeon® E5-2660 v2、2.4 GHz Intel Xeon® E5-2673 v3 (Haswell) 或 2.3 GHz Intel XEON® E5-2673 v4 (Broadwell)  

<br />

### <a name="ds-series---memory-optimized"></a>DS 系列 - 内存优化  

**更新的大小建议** ：[Dsv4 系列](dv4-dsv4-series.md)和 [Ddsv4 系列](ddv4-ddsv4-series.md)

<!--Not Available on [Dav4-series](dav4-dasv4-series.md)-->

ACU：160-250 <sup>1,2</sup>

高级存储：支持

高级存储缓存：支持

| 大小 | vCPU | 内存:GiB | 临时存储 (SSD) GiB | 最大数据磁盘数 | 最大缓存吞吐量和临时存储吞吐量：IOPS/MBps（以 GiB 为单位的缓存大小） | 最大非缓存磁盘吞吐量：IOPS/MBps | 最大 NIC 数/预期网络带宽 (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_DS11 | 2  | 14  | 28  | 8  | 8000/64 (72)    | 6400/64   | 2/1000 |
| Standard_DS12 | 4  | 28  | 56  | 16 | 16000/128 (144) | 12800/128 | 4/2000 |
| Standard_DS13 | 8  | 56  | 112 | 32 | 32000/256 (288) | 25600/256 | 8/4000 |
| Standard_DS14 | 16 | 112 | 224 | 64 | 64000/512 (576) | 51200/512 | 8/8000 |

<sup>1</sup> DS 系列 VM 可能的最大磁盘吞吐量（IOPS 或 MBps）可能受限于附加磁盘的数量、大小和条带化。  有关详细信息，请参阅[高性能设计](premium-storage-performance.md)。
<sup>2</sup> VM 系列可以在下述 CPU 之一上运行：2.2 GHz Intel Xeon® E5-2660 v2、2.4 GHz Intel Xeon® E5-2673 v3 (Haswell) 或 2.3 GHz Intel XEON® E5-2673 v4 (Broadwell)  

<br />

<!--Not Available on ### Ls-series-->
<!--Not Avaialble on ### GS-series -->
<!--Not Available on ### G-series -->
<!--Not Available on ## NV-series -->

## <a name="other-sizes"></a>其他大小

* [常规用途](sizes-general.md)
* [计算优化](sizes-compute.md)
* [内存优化](sizes-memory.md)
    
    <!--Not Available on - [Storage optimized](sizes-storage.md)-->
    
* [GPU](sizes-gpu.md)
    
    <!--Not Available on - [High performance compute](sizes-hpc.md)-->

## <a name="next-steps"></a>后续步骤

了解有关 [Azure 计算单元 (ACU)](acu.md) 如何帮助跨 Azure SKU 比较计算性能的详细信息。

<!-- Update_Description: update meta properties, wording update, update link -->