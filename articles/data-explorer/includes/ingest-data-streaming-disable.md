---
title: include 文件
description: include 文件
author: orspod
ms.service: data-explorer
ms.topic: include
origin.date: 07/13/2020
ms.date: 08/18/2020
ms.author: v-tawe
ms.reviewer: alexefro
ms.custom: include file
ms.openlocfilehash: ea5225779986c2a14f35857c34f67c2f4df884e6
ms.sourcegitcommit: f4bd97855236f11020f968cfd5fbb0a4e84f9576
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/18/2020
ms.locfileid: "91146371"
---
## <a name="disable-streaming-ingestion-on-your-cluster"></a>在群集上禁用流式引入

> [!WARNING]
> 禁用流式引入可能需要几个小时。

在 Azure 数据资源管理器群集上禁用流式引入之前，请从所有相关的表和数据库中删除[流式引入策略](../kusto/management/streamingingestionpolicy.md)。 删除流式引入策略会触发 Azure 数据资源管理器群集内的数据重新排列。 流式引入数据将从初始存储移到列存储中的永久存储（盘区或分片）。 此过程可能需要几秒钟到几个小时，具体取决于初始存储中的数据量。
