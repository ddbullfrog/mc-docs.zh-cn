---
title: 数据分片策略 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的数据分片策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2020
ms.date: 10/29/2020
ms.openlocfilehash: 78a65a42bdf9c123a00bff9e8e6011ab419221f2
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106312"
---
# <a name="data-sharding-policy"></a>数据分片策略

分片策略定义了 Azure 数据资源管理器群集中的[盘区（数据分片）](../management/extents-overview.md)是否应该密封以及如何密封。

> [!NOTE]
> 此策略适用于创建新盘区的所有操作，例如用于[数据引入](../../ingest-data-overview.md#kusto-query-language-ingest-control-commands)的命令以及 [.merge 和 .rebuild 命令](./merge-extents.md)

数据分片策略包含以下属性：

- **MaxRowCount** ：
    - 由引入操作或重新生成操作创建的盘区的最大行计数。
    - 默认值为 750,000。
    - 对[合并操作](mergepolicy.md)无效。
        - 如果必须限制由合并操作创建的盘区中的行数，请在实体的[盘区合并策略](mergepolicy.md)中调整 `RowCountUpperBoundForMerge` 属性。
- **MaxExtentSizeInMb** ：
    - 由合并操作创建的盘区允许的最大压缩数据大小（以 MB 为单位）。
    - 仅对[合并](mergepolicy.md)操作有效。
    - 默认值为 1,024 (1GB)。

- **MaxOriginalSizeInMb** ：
    - 由重新生成操作创建的盘区允许的最大原始数据大小（以 MB 为单位）。
    - 仅对[重新生成](mergepolicy.md)操作有效。
    - 默认值为 2,048 (2GB)。

> [!WARNING]
> 更改数据分片策略之前，请咨询 Azure 数据资源管理器团队。

创建数据库时，它包含默认的数据分片策略。 此策略由数据库中创建的所有表继承（除非在表级别显式重写该策略）。

使用[分片策略控制命令](../management/sharding-policy.md)管理数据库和表的数据分片策略。
