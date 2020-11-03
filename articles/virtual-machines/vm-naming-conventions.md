---
title: Azure VM 大小命名约定
description: 说明用于 Azure VM 大小的命名约定
ms.service: virtual-machines
subservice: sizes
ms.topic: conceptual
origin.date: 07/22/2020
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.custom: sttsinar
ms.openlocfilehash: 876dc7bd48fde8e7a7bf0054c4bc3c41c3d5ef2d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106129"
---
<!--Verify Successfully-->
# <a name="azure-virtual-machine-sizes-naming-conventions"></a>Azure 虚拟机大小命名约定

本页概述了用于 Azure VM 的命名约定。 VM 使用这些命名约定来表示不同的功能和规格。

## <a name="naming-convention-explanation"></a>命名约定说明

**[系列]**  +  **[子系列*]**  +  **[# 个 vCPU]**  +  **[附加功能]**  +  **[加速器类型*]**  +  **[版本]**

|值 | 说明|
|---|---|
| 系列 | 指示 VM 系列| 
| *子系列 | 仅用于专门的 VM 区分|
| # 个 vCPU| 表示 VM 的 vCPU 数目 |
| 累加功能 | 一个或多个小写字母表示累加功能，例如： <br /> a = 基于 AMD 的处理器 <br /> d = 磁盘（本地临时磁盘存在）；<br /> h = 支持休眠 <br /> i = 独立的大小 <br /> l = 内存不足；内存量低于内存密集型大小 <br /> m = 内存密集型；特定大小的最大内存量 <br /> t = 较小内存；特定大小中的最小内存量 <br /> r = 支持 RDMA <br /> s = 支持高级存储（注意：没有 s 属性的一些较新大小仍可支持高级存储，例如 M128、M64 等）<br /> |
| *加速器类型 | 表示专用/GPU SKU 中的硬件加速器的类型。 只有自 2020 年第 3 季度起推出的新的专用/GPU SKU 才能在名称中包含硬件加速器。 |
| 版本 | 表示 VM 系列的版本 |

<!--Not Available on  this is for newer Azure VMs, see [Ddv4 and Ddsv4-series](./ddv4-ddsv4-series.md) -->
<!--Not Available on FEATURE ultra-disk-->

## <a name="example-breakdown"></a>细分示例

**[系列]**  +  **[子系列*]**  +  **[# 个 vCPU]**  +  **[附加功能]**  +  **[加速器类型*]**  +  **[版本]**

<!--Not Available on ### Example 1: M416ms_v2-->
<!--Not Available on ### Example 2: NV16as_v4-->
<!--Not Available on ### Example 3: NC4as_T4_v3-->

## <a name="next-steps"></a>后续步骤

详细了解 Azure 中的可用 [VM 大小](./sizes.md)。

<!-- Update_Description: update meta properties, wording update, update link -->