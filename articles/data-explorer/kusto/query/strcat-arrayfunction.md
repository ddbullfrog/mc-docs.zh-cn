---
title: strcat_array() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 strcat_array()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 536adaaa1f8bd305c7be8edbf4e72d477729c206
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103719"
---
# <a name="strcat_array"></a>strcat_array()

使用指定的分隔符创建一个包含数组值的连接字符串。
    
## <a name="syntax"></a>语法

`strcat_array(`*array* , *delimiter*`)`

## <a name="arguments"></a>参数

* *array* ：一个 `dynamic` 值，表示要连接的值的数组。
* delimeter：一个 `string` 值，将用于连接 array 中的值

## <a name="returns"></a>返回

数组值，连接到一个字符串。

## <a name="examples"></a>示例
  
```kusto
print str = strcat_array(dynamic([1, 2, 3]), "->")
```

|str|
|---|
|1->2->3|