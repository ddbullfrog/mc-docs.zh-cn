---
title: countof() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 countof()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/24/2020
ms.openlocfilehash: 2873e7a9e722f9947b5bc83a16d8dcd50f6e1821
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146493"
---
# <a name="countof"></a>countof()

计算字符串中子字符串的出现次数。 纯字符串匹配项可能重叠；正则表达式匹配项不会重叠。

```kusto
countof("The cat sat on the mat", "at") == 3
countof("The cat sat on the mat", @"\b.at\b", "regex") == 3
```

## <a name="syntax"></a>语法

`countof(`*text*`,` *search* [`,` *kind*]`)`

## <a name="arguments"></a>参数

* *text*：一个字符串。
* *search*：用于在 text 内部匹配的纯字符串或[正则表达式](./re2.md)。
* *kind*：`"normal"|"regex"` 默认值为 `normal`。 

## <a name="returns"></a>返回

搜索字符串可在容器中匹配的次数。 纯字符串匹配项可能重叠；正则表达式匹配项不会重叠。

## <a name="examples"></a>示例

|函数调用|结果|
|---|---
|`countof("aaa", "a")`| 3 
|`countof("aaaa", "aa")`| 3 (不是 2!)
|`countof("ababa", "ab", "normal")`| 2
|`countof("ababa", "aba")`| 2
|`countof("ababa", "aba", "regex")`| 1
|`countof("abcabc", "a.c", "regex")`| 2
    