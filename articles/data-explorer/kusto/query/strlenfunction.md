---
title: strlen() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 strlen()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: f9e7ca67522fe509f96b642118d8358cded36093
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103705"
---
# <a name="strlen"></a>strlen()

返回输入字符串的长度（以字符为单位）。

## <a name="syntax"></a>语法

`strlen(`*source*`)`

## <a name="arguments"></a>参数

* *source* ：将测量字符串长度的源字符串。

## <a name="returns"></a>返回

返回输入字符串的长度（以字符为单位）。

**备注**

字符串中的每个 Unicode 字符都等于 `1`，包括代理项。
（例如，尽管在 UTF-8 编码中需要多个值，但中文字符将被计数一次）。


## <a name="examples"></a>示例

```kusto
print length = strlen("hello")
```

|length|
|---|
|5|

```kusto
print length = strlen("⒦⒰⒮⒯⒪")
```

|length|
|---|
|5|