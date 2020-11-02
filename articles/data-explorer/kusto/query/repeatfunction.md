---
title: repeat() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 repeat()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: b3eefe2fa044e103db774f29caee05093e871d46
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105023"
---
# <a name="repeat"></a>repeat()

生成包含一系列相等值的动态数组。

## <a name="syntax"></a>语法

`repeat(`*value*`,` *count*`)` 

## <a name="arguments"></a>参数

* *value* ：生成数组中元素的值。 值的类型可以是 boolean、integer、long、real、datetime 或 timespan。   
* count：生成数组中的元素计数。 count 必须是整数。
如果 count 等于零，则返回空数组。
如果 count 小于零，则返回 null 值。 

## <a name="examples"></a>示例

以下示例返回 `[1, 1, 1]`：

```kusto
T | extend r = repeat(1, 3)
```