---
title: hash_combine() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 hash_combine()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 11/19/2019
ms.date: 10/29/2020
ms.openlocfilehash: 0fa57dcb3a84abf74e5629eecf1d0edcc7b0bdf4
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103876"
---
# <a name="hash_combine"></a>hash_combine()

合并两个或更多个哈希的哈希值。

## <a name="syntax"></a>语法

`hash_combine(`*h1* `,` *h2* [`,` *h3* ...]`)`

## <a name="arguments"></a>参数

* h1：Long 值，表示第一个哈希值。
* h2：Long 值，表示第二个哈希值。
* hN：Long 值，表示第 N 个哈希值。

## <a name="returns"></a>返回

给定标量的组合哈希值。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print value1 = "Hello", value2 = "World"
| extend h1 = hash(value1), h2=hash(value2)
| extend combined = hash_combine(h1, h2)
```

|value1|value2|h1|h2|已合并|
|---|---|---|---|---|
|你好|World|753694413698530628|1846988464401551951|-1440138333540407281|
