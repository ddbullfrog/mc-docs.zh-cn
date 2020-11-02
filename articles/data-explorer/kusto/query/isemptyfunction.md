---
title: isempty() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 isempty()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 59cc0926eb2bbd2ddca7c59805960edf349967c6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103863"
---
# <a name="isempty"></a>isempty()

如果参数为空字符串或为 null，则返回 `true`。
    
```kusto
isempty("") == true
```

## <a name="syntax"></a>语法

`isempty(`[ *value* ]`)`

## <a name="returns"></a>返回

指示参数是空字符串还是 isnull。

|x|isempty(x)
|---|---
| "" | 是
|"x" | false
|parsejson("")|是
|parsejson("[]")|false
|parsejson("{}")|false

## <a name="example"></a>示例

```kusto
T
| where isempty(fieldName)
| count
```