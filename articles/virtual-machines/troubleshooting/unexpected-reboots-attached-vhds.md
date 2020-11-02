---
title: 排查附加 VHD 的 VM 意外重启
description: 如何排查 VM 的意外重启。
keywords: ssh 连接被拒绝, ssh 错误, azure ssh, SSH 连接失败
services: virtual-machines
author: rockboyfor
manager: digimobile
tags: top-support-issue,azure-service-management,azure-resource-manager
ms.service: virtual-machines
ms.topic: article
origin.date: 11/01/2018
ms.date: 02/10/2020
ms.author: v-yeche
ms.openlocfilehash: b0655515b4914845d1159f3184e5b687db1eca96
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104968"
---
# <a name="troubleshoot-unexpected-reboots-of-vms-with-attached-vhds"></a>排查附加 VHD 的 VM 意外重启

如果 Azure 虚拟机 (VM) 具有大量位于同一存储帐户的附加 VHD，则可能会超过单个存储帐户的可伸缩性目标，从而导致 VM 意外重启。 检查存储帐户的分钟指标 ( **TotalRequests**/**TotalIngress**/**TotalEgress** )，了解超过存储帐户的可伸缩性目标的峰值。 若要帮助确定是否已对存储帐户进行限制，请参阅[指标显示 PercentThrottlingError 增加](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md#metrics-show-an-increase-in-PercentThrottlingError)。

通常，虚拟机对 VHD 进行的每个单独的输入或输出操作都会转换为对基础页 Blob 进行的“Get 页”  或“Put 页”  操作。 因此，可以根据应用程序的特定行为，对环境使用估计的 IOPS 以优化可以在单个存储帐户中设置的 VHD 数。 Azure 建议在单个存储帐户中设置 40 个或更少的磁盘。 有关标准存储帐户的可伸缩性目标的详细信息，请参阅[标准存储帐户的可伸缩性目标](../../storage/common/scalability-targets-standard-account.md)。 若要详细了解高级页 Blob 存储帐户的可伸缩性目标，请参阅[高级页 Blob 存储帐户的可伸缩性目标](../../storage/blobs/scalability-targets-premium-page-blobs.md)。

如果即将超过存储帐户的可伸缩性目标，请将 VHD 放入多个存储帐户中，以减少每个帐户中的活动。

<!-- Update_Description: update meta properties, wording update, update link -->