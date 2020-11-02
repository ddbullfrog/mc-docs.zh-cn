---
title: string_size() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 string_size()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 45f307cc824abfee8b3bcd9a0ab59a39d72534e4
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103706"
---
# <a name="string_size"></a>string_size()

返回输入字符串的大小（以字节为单位）。

## <a name="syntax"></a>语法

`string_size(`*source*`)`

## <a name="arguments"></a>参数

* *source* ：将测量字符串大小的源字符串。

## <a name="returns"></a>返回

返回输入字符串的长度（以字节为单位）。

## <a name="examples"></a>示例

```kusto
print size = string_size("hello")
```

|大小|
|---|
|5|

```kusto
print size = string_size("⒦⒰⒮⒯⒪")
```

|大小|
|---|
|15|