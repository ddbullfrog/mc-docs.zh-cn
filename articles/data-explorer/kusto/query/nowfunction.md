---
title: now() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 now()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 4fe950b389f7d12f1a637684bb6cde504167fa26
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103853"
---
# <a name="now"></a>now()

返回当前的 UTC 时钟时间，偏移量取决于给定的时间跨度（可选）。
此函数可在语句中多次使用，并且所有实例引用的时钟时间均相同。

```kusto
now()
now(-2d)
```

## <a name="syntax"></a>语法

`now(`[ *offset* ]`)`

## <a name="arguments"></a>参数

* *offset* ：`timespan`，已添加到当前的 UTC 时钟时间。 默认值：0。

## <a name="returns"></a>返回

当前 UTC 时钟时间，用作 `datetime`。

`now()` + *offset* 

## <a name="example"></a>示例

确定 predicate 标识事件后的时间间隔：

```kusto
T | where ... | extend Elapsed=now() - Timestamp
```