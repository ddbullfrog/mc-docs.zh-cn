---
title: Azure Batch 分析
description: 批处理分析中的主题包含可用于批处理服务资源的事件和警报的参考信息。
ms.topic: reference
origin.date: 10/08/2020
author: rockboyfor
ms.date: 11/02/2020
ms.author: v-yeche
ms.openlocfilehash: 9a4d360bb3ff8eef7e0cf6d92b964f3977c41ad8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104953"
---
# <a name="batch-analytics"></a>批处理分析

本部分中的主题包含可用于批处理服务资源的事件和警报的参考信息。

有关启用和使用批处理诊断日志的详细信息，请参阅 [Azure Batch 诊断日志记录](./batch-diagnostics.md)。

## <a name="diagnostic-logs"></a>诊断日志

Azure Batch 服务会在某些批处理资源的生命周期内生成以下诊断日志事件。

### <a name="service-log-events"></a>服务日志事件

- [池创建](batch-pool-create-event.md)
- [池删除启动](batch-pool-delete-start-event.md)
- [池删除完成](batch-pool-delete-complete-event.md)
- [池调整大小启动](batch-pool-resize-start-event.md)
- [池调整大小完成](batch-pool-resize-complete-event.md)
- [池自动缩放](batch-pool-autoscale-event.md)
- [任务启动](batch-task-start-event.md)
- [任务完成](batch-task-complete-event.md)
- [任务失败](batch-task-fail-event.md)
- [任务计划失败](batch-task-schedule-fail-event.md)

<!-- Update_Description: update meta properties, wording update, update link -->