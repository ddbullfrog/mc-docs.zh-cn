---
title: parse_user_agent() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 parse_user_agent()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2020
ms.date: 10/29/2020
ms.openlocfilehash: da9efaf93605041a01540ed4865830efbcdb396b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106147"
---
# <a name="parse_user_agent"></a>parse_user_agent()

解释一个 user-agent 字符串，该字符串用于标识用户的浏览器，并向承载用户所访问网站的服务器提供特定的系统详细信息。 结果以 [`dynamic`](./scalar-data-types/dynamic.md) 的形式返回。 

## <a name="syntax"></a>语法

`parse_user_agent(`*user-agent-string* , *look-for*`)`

## <a name="arguments"></a>参数

* *user-agent-string* ：`string` 类型的表达式，表示 user-agent 字符串。

* *look-for* ：`string` 或 `dynamic` 类型的表达式，表示该函数在 user-agent 字符串（分析目标）中应该查找的内容。 可能的选项：“browser”、“os”、“device”。 如果只需要一个分析目标，可将 `string` 参数传递给它。
如果需要两个或三个，则可以将其作为 `dynamic array` 传递。

## <a name="returns"></a>返回

`dynamic` 类型的对象，包含有关所请求的分析目标的信息。

Browser：Family, MajorVersion, MinorVersion, Patch                 

OperatingSystem:Family, MajorVersion, MinorVersion, Patch, PatchMinor             

设备：Family, Brand, Model

> [!WARNING]
> 该函数实现以输入字符串的正则表达式检查（针对海量预定义模式）为基础。 因此，时间和 CPU 用量预计会很高。
在查询中使用该函数时，请确保它以分布式方式在多台计算机上运行。
如果频繁使用带有此函数的查询，那么你可能想要通过[更新策略](../management/updatepolicy.md)预创建结果，但需考虑到，在更新策略中使用此函数将会增加引入延迟。
 
## <a name="example"></a>示例

```kusto
print useragent = "Mozilla/5.0 (Windows; U; en-US) AppleWebKit/531.9 (KHTML, like Gecko) AdobeAIR/2.5.1"
| extend x = parse_user_agent(useragent, "browser") 
```

预期结果是动态对象：

{ "Browser": { "Family":"AdobeAIR", "MajorVersion":"2", "MinorVersion":"5", "Patch":"1" } }

```kusto
print useragent = "Mozilla/5.0 (SymbianOS/9.2; U; Series60/3.1 NokiaN81-3/10.0.032 Profile/MIDP-2.0 Configuration/CLDC-1.1 ) AppleWebKit/413 (KHTML, like Gecko) Safari/4"
| extend x = parse_user_agent(useragent, dynamic(["browser","os","device"])) 
```

预期结果是动态对象：

{ "Browser": { "Family":"Nokia OSS Browser", "MajorVersion":"3", "MinorVersion":"1", "Patch": "" }, "OperatingSystem": { "Family":"Symbian OS", "MajorVersion":"9", "MinorVersion":"2", "Patch": "", "PatchMinor": "" }, "Device": { "Family":"Nokia N81", "Brand":"Nokia", "Model":"N81-3" } }