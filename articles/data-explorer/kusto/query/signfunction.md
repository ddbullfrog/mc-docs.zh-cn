---
title: sign() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 sign()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: a644dd26b50399e5a78bfdafab6ebd66ad6b20cb
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104261"
---
# <a name="sign"></a>sign()

数值表达式的符号

## <a name="syntax"></a>语法

`sign(`*x*`)`

## <a name="arguments"></a>参数

* x：一个实数。

## <a name="returns"></a>返回

* 指定表达式的正号 (+1)、零 (0) 或负号 (-1)。 

## <a name="examples"></a>示例

```kusto
print s1 = sign(-42), s2 = sign(0), s3 = sign(11.2)

```

|s1|s2|s3|
|---|---|---|
|-1|0|1|