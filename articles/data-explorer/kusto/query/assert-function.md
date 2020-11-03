---
title: assert() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 assert()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 09/26/2019
ms.date: 10/29/2020
ms.openlocfilehash: a87338c730cd0e5303845b58b01f2e318bedd74e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106309"
---
# <a name="assert"></a>assert()

检查条件。 如果条件为 false，将输出错误消息并使查询失败。

## <a name="syntax"></a>语法

`assert(`*condition*`, `*message*`)`

## <a name="arguments"></a>参数

* *condition* ：要计算的条件表达式。 如果条件为 `false`，则使用指定的消息报告错误。 如果条件为 `true`，它将返回 `true` 作为计算结果。 在查询分析阶段，条件的计算结果必须为常量。
* *message* ：断言计算结果为 `false` 时使用的消息。 message 必须是字符串字文本。

> [!NOTE]
> 在查询分析阶段，`condition` 的计算结果必须为常量。 换句话说，可以从引用常量的其他表达式进行构造它，而不能将其绑定到行上下文。

## <a name="returns"></a>返回

* `true` - 如果条件为 `true`
* 如果条件计算结果为 `false`，则引发语义错误。

## <a name="examples"></a>示例

下面的查询定义了用于检查输入字符串长度的函数 `checkLength()`，并使用 `assert` 验证输入长度参数（检查它是否大于零）。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let checkLength = (len:long, s:string)
{
    assert(len > 0, "Length must be greater than zero") and 
    strlen(s) > len
};
datatable(input:string)
[
    '123',
    '4567'
]
| where checkLength(len=long(-1), input)
```

运行此查询将产生错误：  
`assert() has failed with message: 'Length must be greater than zero'`


使用有效的 `len` 输入运行的示例：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let checkLength = (len:long, s:string)
{
    assert(len > 0, "Length must be greater than zero") and strlen(s) > len
};
datatable(input:string)
[
    '123',
    '4567'
]
| where checkLength(len=3, input)
```

|input|
|---|
|4567|
