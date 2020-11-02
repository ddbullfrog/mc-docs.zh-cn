---
title: strcat_delim() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 strcat_delim()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 13b3e8357a2911e03ba1917c188fb2ca6c58b33a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103717"
---
# <a name="strcat_delim"></a>strcat_delim()

连接 2 到 64 个参数（分隔符作为第一个参数提供）。

 * 如果参数不是字符串类型，则会将它们强制转换为字符串。

## <a name="syntax"></a>语法

`strcat_delim(`*delimiter* , *argument1* , *argument2* [ , *argumentN* ]`)`

## <a name="arguments"></a>参数

* delimiter：要用作分隔符的字符串表达式。
* argument1 ... argumentN：要连接的表达式。

## <a name="returns"></a>返回

参数（已使用 delimiter 连接到单个字符串）。

## <a name="examples"></a>示例

```kusto
print st = strcat_delim('-', 1, '2', 'A', 1s)

```

|st|
|---|
|1-2-A-00:00:01|
