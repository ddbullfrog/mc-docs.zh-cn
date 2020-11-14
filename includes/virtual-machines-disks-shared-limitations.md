---
title: include 文件
description: include 文件
services: virtual-machines
ms.service: virtual-machines
ms.topic: include
origin.date: 09/30/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: yes|no
ms.testdate: 11/09/2020null
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: 79b90d4d0568ee258b7334f0d14ce6a30177b538
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328992"
---
<!--Verified successfully for PG notification-->
仅可对部分磁盘类型启用共享磁盘。 目前只有高级 SSD 可启用共享磁盘。 已启用共享磁盘的托管磁盘受到下列限制，这些限制按磁盘类型进行整理：

<!--Not Available on FEATURE ultra disks-->

<!--Not Available on ### Ultra disks-->

### <a name="premium-ssds"></a>高级 SSD

- 当前仅限于 Azure 资源管理器或 SDK 支持。 
- 仅可对数据磁盘启用，不可对 OS 磁盘启用。
- ReadOnly 主机缓存不适用于采用 `maxShares>1` 的高级 SSD。
- 磁盘突发不适用于采用 `maxShares>1` 的高级 SSD。
- 通过 Azure 共享磁盘将可用性集与虚拟机规模集一起使用时，不会对共享数据磁盘强制实施与虚拟机容错域的[存储容错域对齐](https://docs.azure.cn/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set)。
- 使用 [邻近放置组 (PPG)](../articles/virtual-machines/windows/proximity-placement-groups.md) 时，共享一个磁盘的所有虚拟机都必须属于同一个 PPG。
- 只可对 Windows Server 故障转移群集的某些版本使用基本磁盘；有关详细信息，请参阅[故障转移群集硬盘要求和存储选项](https://docs.microsoft.com/windows-server/failover-clustering/clustering-requirements)。
- Azure 备份和 Azure Site Recovery 支持尚不可用。

#### <a name="regional-availability"></a>区域可用性

可在提供托管磁盘的所有区域中使用共享高级 SSD。

<!-- Update_Description: new article about virtual machines disks shared limitations -->
<!--NEW.date: 11/09/2020-->