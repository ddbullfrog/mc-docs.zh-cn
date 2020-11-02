---
title: strcat() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 strcat()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 0c44ca1a4e78d649ce1fbca8926bef9722c20598
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103715"
---
# <a name="strcat"></a>strcat()

连接 1 到 64 个参数。

* 如果参数不是字符串类型，则会将它们强制转换为字符串。

## <a name="syntax"></a>语法

`strcat(`*argument1* , *argument2* [, *argumentN* ]`)`

## <a name="arguments"></a>参数

* argument1 ... argumentN：要连接的表达式。

## <a name="returns"></a>返回

参数（已连接到一个字符串）。

## <a name="examples"></a>示例
  
   ```kusto
print str = strcat("hello", " ", "world")
```

|str|
|---|
|hello world|
