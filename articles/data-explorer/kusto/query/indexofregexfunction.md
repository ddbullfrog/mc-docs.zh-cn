---
title: indexof_regex() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 indexof_regex()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 7c4defe020d4020471ab39acaace7b435e603a43
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104385"
---
# <a name="indexof_regex"></a>indexof_regex()

函数会报告输入字符串中指定的字符串第一次出现时的索引（从零开始）。 纯字符串匹配项不重叠。

请参阅 [`indexof()`](indexoffunction.md)。

## <a name="syntax"></a>语法

`indexof_regex(`*source*`,`*lookup*`[,`*start_index*`[,`*length*`[,`*occurrence*`]]])`

## <a name="arguments"></a>自变量

|参数     | 描述                                     |必需还是可选|
|--------------|-------------------------------------------------|--------------------|
|source        | 输入字符串                                    |必须            |
|查找        | 要查找的字符串                                  |必须            |
|start_index   | 搜索开始位置                           |可选            |
|length        | 要检查的字符位置数。 -1 定义无限长度 |可选            |
|occurrence    | 查找模式第 N 次出现时的索引。 
                 默认值为 1（第一次出现时的索引） |可选            |

## <a name="returns"></a>返回

查找的从零开始的索引位置。

* 如果在输入中找不到该字符串，则返回 -1。
* 在以下情况下返回 null：
     * start_index 小于 0。
     * 出现次数小于 0。
     * 长度参数小于 -1。


## <a name="examples"></a>示例

```kusto
print
 idx1 = indexof_regex("abcabc", "a.c") // lookup found in input string
 , idx2 = indexof_regex("abcabcdefg", "a.c", 0, 9, 2)  // lookup found in input string
 , idx3 = indexof_regex("abcabc", "a.c", 1, -1, 2)  // there is no second occurrence in the search range
 , idx4 = indexof_regex("ababaa", "a.a", 0, -1, 2)  // Plain string matches do not overlap so full lookup can't be found
 , idx5 = indexof_regex("abcabc", "a|ab", -1)  // invalid input
```

|idx1|idx2|idx3|idx4|idx5|
|----|----|----|----|----|
|0   |3   |-1  |-1  |    |
