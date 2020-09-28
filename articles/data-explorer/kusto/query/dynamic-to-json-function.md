---
title: dynamic_to_json() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 dynamic_to_json()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: elgevork
ms.service: data-explorer
ms.topic: reference
origin.date: 08/05/2020
ms.date: 09/24/2020
ms.openlocfilehash: dbe9a5b62a366a65d4df9e8ea7fb7514eb5f5259
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146597"
---
# <a name="dynamic_to_json"></a>dynamic_to_json()

将 `dynamic` 输入转换为字符串表示形式。
如果输入是属性包，则输出字符串会以递归方式输出其按照键排序的内容。 否则，输出与 `tostring` 函数输出类似。

## <a name="syntax"></a>语法

`dynamic_to_json(Expr)`

## <a name="arguments"></a>参数

* Expr：`dynamic` 输入。 该函数接受一个参数。

## <a name="returns"></a>返回

返回 `dynamic` 输入的字符串表示形式。 如果输入是属性包，则输出字符串会以递归方式输出其按照键排序的内容。

## <a name="examples"></a>示例

表达式：

```kusto
  let bag1 = dynamic_to_json(dynamic({ 'Y10':dynamic({ }), 'X8': dynamic({ 'c3':1, 'd8':5, 'a4':6 }),'D1':114, 'A1':12, 'B1':2, 'C1':3, 'A14':[15, 13, 18]}));
  print bag1
```
  
结果：

```
"{
  ""A1"": 12,
  ""A14"": [
    15,
    13,
    18
  ],
  ""B1"": 2,
  ""C1"": 3,
  ""D1"": 114,
  ""X8"": {
    ""c3"": 1,
    ""d8"": 5,
    ""a4"": 6
  },
  ""Y10"": {}
}"
```

表达式：

```kusto
 let bag2 = dynamic_to_json(dynamic({ 'X8': dynamic({ 'a4':6, 'c3':1, 'd8':5}), 'A14':[15, 13, 18], 'C1':3, 'B1':2, 'Y10': dynamic({ }), 'A1':12, 'D1':114}));
 print bag2
```
 
结果：

```
{
  "A1": 12,
  "A14": [
    15,
    13,
    18
  ],
  "B1": 2,
  "C1": 3,
  "D1": 114,
  "X8": {
    "a4": 6,
    "c3": 1,
    "d8": 5
  },
  "Y10": {}
}
```
