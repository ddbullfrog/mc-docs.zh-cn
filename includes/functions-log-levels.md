---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 10/19/2020
ms.author: v-junlch
ms.openlocfilehash: 8ac2d2b1a4e8eafe19125b2f98c4baf3b18b7f3d
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472053"
---
为每个日志分配日志级别。 该值是表示相对重要性的整数：

|LogLevel    |代码| 说明 |
|------------|---|--------------|
|跟踪       | 0 |包含最详细消息的日志。 这些消息可能包含敏感应用程序数据。 这些消息默认情况下处于禁用状态，并且绝不应在生产环境中启用。|
|调试       | 1 | 在开发过程中用于交互式调查的日志。 这些日志应主要包含对调试有用的信息，并且没有长期价值。 |
|信息 | 2 | 跟踪应用程序的常规流的日志。 这些日志应具有长期价值。 |
|警告     | 3 | 突出显示应用程序流中的异常或意外事件，但不会导致应用程序执行停止的日志。 |
|错误       | 4 | 当前执行流因失败而停止时突出显示的日志。 这些错误应指示当前活动中的故障，而不是应用程序范围内的故障。 |
|严重    | 5 | 描述不可恢复的应用程序/系统崩溃或需要立即引起注意的灾难性故障的日志。 |
|无        | 6 | 禁用指定类别的日志记录。 |

[Host.json 文件](../articles/azure-functions/functions-host-json.md)配置确定函数应用发送到 Application Insights 的日志记录数量。  

