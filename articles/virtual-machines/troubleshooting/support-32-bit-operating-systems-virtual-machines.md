---
title: Azure 虚拟机对 32 位操作系统的支持 | Azure
description: 有关 Azure 虚拟机支持的操作系统的信息
services: virtual-machines-windows, azure-resource-manager
manager: dcscontentpm
tags: top-support-issue, azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
origin.date: 09/18/2019
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 0097015dc9d7be3b17dea53c9e496b854a731254
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104978"
---
# <a name="support-for-32-bit-operating-systems-in-azure-virtual-machines"></a>Azure 虚拟机对 32 位操作系统的支持

Azure 现在允许用户将其 32 位 Windows 操作系统引入 Azure。 仅支持专用 VHD，通用化映像在 Azure 中无效。 由于其中一些操作系统的生命周期可支持协议已结束，因此 Azure 可能不会为它们提供额外的支持。 也不提供对在 Azure 虚拟机 (VM) 上运行的基于 Linux 或基于 Berkeley Software Distribution (BSD) 的操作系统的支持。

## <a name="more-information"></a>详细信息

有关 Azure 虚拟机支持的操作系统的详细信息，请参阅以下 Microsoft 知识库文章：

<!--MOONCAKE: CORRECT ON Microsoft Knowledge Base articles-->

* [Microsoft 服务器软件对 Azure 虚拟机的支持](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines)
* [Azure 对 Linux 和开放源代码技术的支持](https://support.microsoft.com/help/2941892/support-for-linux-and-open-source-technology-in-azure)

## <a name="references"></a>参考

* [详细了解 Azure 中 Windows Server 2008/R2 的免费扩展安全更新](https://www.microsoft.com/cloud-platform/windows-server-2008)
* [详细了解 Azure 对 Windows Server 2008 SP2 32 位专用映像的支持](https://docs.microsoft.com/windows-server/get-started/uploading-specialized-ws08-image-to-azure)
* [详细了解对使用 Azure Site Recovery 将 Windows Server 2008 映像迁移到 Azure 的支持](../../site-recovery/migrate-tutorial-windows-server-2008.md)
* [详细了解 Azure 扩展支持的操作系统](https://support.microsoft.com/help/4078134/azure-extension-supported-operating-systems)
* [详细了解如何在 Azure 上运行 Windows Server 2003](https://support.microsoft.com/help/3206074/running-windows-server-2003-on-microsoft-azure)

## <a name="next-steps"></a>后续步骤

如果对本文中的任何观点存在疑问，请联系 [Azure 支持](https://support.azure.cn/support/contact/)上的 Azure 专家。

或者，提交 Azure 支持事件。 请转到 [Azure 支持站点](https://support.azure.cn/support/support-azure/)提交请求。

<!-- Update_Description: update meta properties, wording update, update link -->
