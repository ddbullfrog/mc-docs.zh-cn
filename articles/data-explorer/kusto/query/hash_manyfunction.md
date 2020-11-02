---
title: hash_many() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 hash_many()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/06/2020
ms.date: 10/29/2020
ms.openlocfilehash: 382fc789a430423f149729711b3121c18ae9ff75
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103879"
---
# <a name="hash_many"></a>hash_many()

返回多个值的组合哈希值。

## <a name="syntax"></a>语法

`hash_many(`*s1* `,` *s2* [`,` *s3* ...]`)`

## <a name="arguments"></a>参数

* s1、s2、...、sN：将一起进行哈希处理的输入值。

## <a name="returns"></a>返回

给定标量的组合哈希值。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print value1 = "Hello", value2 = "World"
| extend combined = hash_many(value1, value2)
```

|value1|value2|已合并|
|---|---|---|
|你好|World|-1440138333540407281|
