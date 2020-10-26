---
author: ggailey777
ms.author: v-junlch
ms.date: 10/19/2020
ms.topic: include
ms.service: azure-functions
ms.openlocfilehash: 816022dc1ed34ef6a430033b44bf0f0b433bafdf
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92471475"
---
#### <a name="built-in-log-streaming"></a>内置日志流式处理

使用 `logstream` 选项开始接收在 Azure 中运行的特定函数应用的流式处理日志，如以下示例所示：

```bash
func azure functionapp logstream <FunctionAppName>
```

#### <a name="live-metrics-stream"></a>实时指标流

通过包含 `--browser` 选项，可在新的浏览器窗口中查看函数应用的[实时指标流](../articles/azure-monitor/app/live-stream.md)，如下例所示：

```bash
func azure functionapp logstream <FunctionAppName> --browser
```

