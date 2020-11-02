---
title: ago() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 ago()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: eb82a9653f578fadbf709fb05d49d81c3c7a2973
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105059"
---
# <a name="ago"></a>ago()

从当前 UTC 时钟时间减去给定时间跨度。

```kusto
ago(1h)
ago(1d)
```

与 `now()` 类似，此函数可在语句中多次使用，并且所有实例化引用的 UTC 时钟时间均相同。

## <a name="syntax"></a>语法

`ago(`a_timespan`)`

## <a name="arguments"></a>参数

* a_timespan：要从当前 UTC 时钟时间 (`now()`) 减去的间隔。

## <a name="returns"></a>返回

`now() - a_timespan`

## <a name="example"></a>示例

过去一小时内具有时间戳的所有行：

```kusto
T | where Timestamp > ago(1h)
```