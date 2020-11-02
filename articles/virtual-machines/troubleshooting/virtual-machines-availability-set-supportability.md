---
title: 将 Azure VM 添加到现有可用性集的可支持性 | Azure
description: 本文提供了一个可支持性矩阵，说明可以在同一可用性集中混合哪些 VM 系列
author: rockboyfor
manager: digimobile
ms.service: virtual-machines
ms.topic: troubleshooting
origin.date: 06/15/2018
ms.date: 04/27/2020
ms.author: v-yeche
ms.openlocfilehash: 5ce4c5ea980e7607a68cf6301cf6b8e274a618f0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105241"
---
# <a name="supportability-of-adding-azure-vms-to-an-existing-availability-set"></a>将 Azure VM 添加到现有可用性集的可支持性

将新的虚拟机 (VM) 添加到现有可用性集时，可能偶尔会遇到限制。 以下图表详细说明了可以在同一可用性集中混合的 VM 系列。

以下是混合不同类型的 VM 的可支持性矩阵：

系列和可用性集|第二个 VM|A|Av2|D|Dv2|Dv3|
|---|---|---|---|---|---|---|
|第一个 VM|||||||
|A||OK|OK|OK|OK|OK|
|Av2||OK|OK|OK|OK|OK|
|D||OK|OK|OK|OK|OK|
|Dv2||OK|OK|OK|OK|OK|
|Dv3||OK|OK|OK|OK|OK|

所有其他系列都不能在同一可用性集中，因为它们需要特定的硬件。

由于专用 RDMA 后端网络的要求，不能混合使用 A8/A9 VM 大小。

<!-- Update_Description: update meta properties, wording update, update link -->