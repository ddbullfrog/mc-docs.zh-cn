---
title: to_utf8() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 to_utf8()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: faea27949f315d2b818b77bbecdb20f6f61f3f43
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105091"
---
# <a name="to_utf8"></a>to_utf8()

返回输入字符串的 unicode 字符的动态数组（make_string 的反运算）。

## <a name="syntax"></a>语法

`to_utf8(`*source*`)`

## <a name="arguments"></a>参数

* *source* ：要转换的源字符串。

## <a name="returns"></a>返回

返回由 unicode 字符组成的动态数组，这些字符组成提供给此函数的字符串。
请参阅 [`make_string()`](makestringfunction.md)。

## <a name="examples"></a>示例

```kusto
print arr = to_utf8("⒦⒰⒮⒯⒪")
```

|arr|
|---|
|[9382, 9392, 9390, 9391, 9386]|

```kusto
print arr = to_utf8("קוסטו - Kusto")
```

|arr|
|---|
|[1511, 1493, 1505, 1496, 1493, 32, 45, 32, 75, 117, 115, 116, 111]|

```kusto
print str = make_string(to_utf8("Kusto"))
```

|str|
|---|
|Kusto|
