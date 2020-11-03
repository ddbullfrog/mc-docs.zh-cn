---
title: log() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 log()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 08/11/2019
ms.date: 10/29/2020
ms.openlocfilehash: 5cd14b615c5142c4bddf2dcf901ef3d5779a4033
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105925"
---
# <a name="log"></a>log()

`log()` 返回自然对数函数。  

## <a name="syntax"></a>语法

`log(`*x*`)`

## <a name="arguments"></a>参数

* x：大于 0 的实数。

## <a name="returns"></a>返回

* 自然对数是以 e 为底的对数：自然指数函数 (exp) 的反函数。
* 如果参数为负或为 NULL 或不能转换为 `real` 值，则此项为 `null`。 

## <a name="see-also"></a>请参阅

* 有关常用对数（以 10 为底），请参阅 [log10()](log10-function.md)。
* 有关以 2 为底的对数，请参阅 [log2()](log2-function.md)