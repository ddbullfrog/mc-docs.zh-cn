---
title: 正则表达式 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的正则表达式。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 12/09/2019
ms.date: 09/30/2020
ms.openlocfilehash: f989a0fe0555d1b785d454a9e099eeb9b09e7a33
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104003"
---
# <a name="re2-syntax"></a>RE2 语法

RE2 正则表达式语法描述 Kusto (re2) 使用的正则表达式库的语法。
Kusto 中有一些函数使用正则表达式执行字符串匹配、选择和提取

- [countof()](countoffunction.md)
- [extract()](extractfunction.md)
- [extract_all()](extractallfunction.md)
- [matches regex](datatypes-string-operators.md)
- [parse 运算符](parseoperator.md)
- [replace()](replacefunction.md)
- [trim()](trimfunction.md)
- [trimend()](trimendfunction.md)
- [trimstart()](trimstartfunction.md)

Kusto 支持的正则表达式语法是 [re2 库](https://github.com/google/re2/wiki/Syntax)的语法。 这些表达式必须在 Kusto 中编码为 `string` 文本，Kusto 的所有字符串引用规则都适用。 例如，正则表达式 `\A` 匹配一行的开头，并且在 Kusto 中指定为字符串文本 `"\\A"`（请注意“额外的”反斜杠 (`\`) 字符）。
