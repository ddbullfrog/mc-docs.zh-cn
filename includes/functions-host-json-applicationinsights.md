---
title: include 文件
description: include 文件
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 10/19/2020
ms.author: v-junlch
ms.custom: include file
ms.openlocfilehash: 627ec32096fd47a5b8b7aa40227c2ae518450fb2
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472583"
---
控制 [Application Insights 中的采样功能](../articles/azure-functions/configure-monitoring.md#configure-sampling)。

```json
{
    "applicationInsights": {
        "sampling": {
          "isEnabled": true,
          "maxTelemetryItemsPerSecond" : 5
        }
    }
}
```

|属性  |默认 | 说明 |
|---------|---------|---------| 
|isEnabled|true|启用或禁用采样。| 
|maxTelemetryItemsPerSecond|5|开始采样所要达到的阈值。| 

