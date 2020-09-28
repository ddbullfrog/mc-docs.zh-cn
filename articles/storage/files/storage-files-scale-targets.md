---
title: Azure 文件可伸缩性和性能目标
description: 了解 Azure 文件的可伸缩性和性能目标信息，包括容量、请求速率以及入站和出站带宽限制。
author: WenJason
ms.service: storage
ms.topic: conceptual
origin.date: 10/16/2019
ms.date: 09/28/2020
ms.author: v-jay
ms.subservice: files
ms.openlocfilehash: 3e3a846dcd22770d0cf6cdbd9b83cbba44a05b7b
ms.sourcegitcommit: 119a3fc5ffa4768b1bd8202191091bd4d873efb4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2020
ms.locfileid: "91026586"
---
# <a name="azure-files-scalability-and-performance-targets"></a>Azure 文件可伸缩性和性能目标

[Azure 文件](storage-files-introduction.md)在云中提供完全托管的文件共享，这些共享项可通过行业标准 SMB 协议进行访问。 本文讨论了 Azure 文件的可伸缩性和性能目标。

此处列出的可伸缩性和性能目标是高端目标，但可能会受部署中的其他变量影响。 例如，除了受限于托管着 Azure 文件服务的服务器之外，针对文件的吞吐量还可能会受限于可变的网络带宽。 强烈建议你对使用模式进行测试，以确定 Azure 文件的可伸缩性和性能是否满足你的要求。 随着时间的推移，我们也一直在努力提高这些限制。 

## <a name="azure-storage-account-scale-targets"></a>Azure 存储帐户规模目标

Azure 文件共享的父资源是 Azure 存储帐户。 存储帐户表示 Azure 中的一个存储池，该存储池可供包括 Azure 文件在内多个存储服务用来存储数据。 在存储帐户中存储数据的其他服务有 Azure Blob 存储、Azure 队列存储和 Azure 表存储。 以下目标适用于在存储帐户中存储数据的所有存储服务：

[!INCLUDE [azure-storage-account-limits-standard](../../../includes/azure-storage-account-limits-standard.md)]

[!INCLUDE [azure-storage-limits-azure-resource-manager](../../../includes/azure-storage-limits-azure-resource-manager.md)]

> [!Important]  
> 其他存储服务的常规用途存储帐户利用率会影响存储帐户中的 Azure 文件共享。 例如，如果由于 Azure Blob 存储而达到了最大存储帐户容量，则将无法在 Azure 文件共享上创建新文件，即使 Azure 文件共享低于最大共享大小。

## <a name="azure-files-scale-targets"></a>Azure 文件规模目标

对于 Azure 文件存储，需要考虑三类限制：存储帐户、共享和文件。

例如：使用高级文件共享时，单个共享可以达到 100,000 IOPS，单个文件最多可以扩展到 5,000 IOPS。 因此，如果一个共享中有三个文件，则可以从该共享获取的最大 IOPS 为 15,000。

### <a name="standard-storage-account-limits"></a>标准存储帐户限制

有关这些限制，请参阅 [Azure 存储帐户规模目标](#azure-storage-account-scale-targets)部分。

### <a name="premium-filestorage-account-limits"></a>高级 FileStorage 帐户限制

[!INCLUDE [azure-storage-limits-filestorage](../../../includes/azure-storage-limits-filestorage.md)]

> [!IMPORTANT]
> 存储帐户限制适用于所有共享。 仅当每个 FileStorage 帐户只有一个共享时，才能实现 FileStorage 帐户的最大扩展。

### <a name="file-share-and-file-scale-targets"></a>文件共享和文件缩放目标

> [!NOTE]
> 超过 5 TiB 的标准文件共享具有某些限制。 有关启用较大文件共享大小的限制和说明的列表，请参阅规划指南的[在标准文件共享上启用较大文件共享](storage-files-planning.md#enable-standard-file-shares-to-span-up-to-100-tib)部分。

[!INCLUDE [storage-files-scale-targets](../../../includes/storage-files-scale-targets.md)]

[!INCLUDE [storage-files-premium-scale-targets](../../../includes/storage-files-premium-scale-targets.md)]

## <a name="see-also"></a>另请参阅

- [规划 Azure 文件部署](storage-files-planning.md)
