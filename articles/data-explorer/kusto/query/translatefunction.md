---
title: translate() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 translate()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/11/2019
ms.date: 09/30/2020
ms.openlocfilehash: 5bb6480496e0ab0e53c324d0fb75d646ab16bd14
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105087"
---
# <a name="translate"></a>translate()

将一组字符（“searchList”）替换为给定字符串中的另一组字符（“replacementList”）。
此函数在“searchList”中搜索字符，并将其替换为“replacementList”中的相应字符

## <a name="syntax"></a>语法

`translate(`*searchList*`,` *replacementList*`,` *text*`)`

## <a name="arguments"></a>参数

* searchList：应被替换的字符的列表
* replacementList：应替换“searchList”中字符的字符的列表
* *text* ：要搜索的字符串

## <a name="returns"></a>返回

将“replacementList”中出现的所有字符替换为“searchList”中的相应字符后得到的 text

## <a name="examples"></a>示例

|输入                                 |输出   |
|--------------------------------------|---------|
|`translate("abc", "x", "abc")`        |`"xxx"`  |
|`translate("abc", "", "ab")`          |`""`     |
|`translate("krasp", "otsku", "spark")`|`"kusto"`|