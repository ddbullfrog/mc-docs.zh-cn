---
title: 数据更改 - LUIS
description: 了解如何在语言理解 (LUIS) 得出预测之前更改数据
ms.service: cognitive-services
ms.subservice: language-understanding
ms.author: v-johya
ms.topic: conceptual
ms.date: 10/19/2020
origin.date: 02/11/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 784e7b630fa29e67a1582757d8a93962b8eff6d6
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472497"
---
# <a name="alter-utterance-data-before-or-during-prediction"></a>在预测之前或预测期间更改话语数据
LUIS 提供在预测之前或预测期间操作陈述的方法。 这些方法包括修复预生成 [datetimeV2](luis-reference-prebuilt-datetimev2.md) 的时区问题。
<!--Not available in MC: ## Correct spelling errors in utterance-->

## <a name="change-time-zone-of-prebuilt-datetimev2-entity"></a>更改预生成 datetimeV2 实体的时区
LUIS 应用使用预生成的 [datetimeV2](luis-reference-prebuilt-datetimev2.md) 实体时，可以在预测响应中返回日期/时间值。 请求的时区用于确定要返回的正确日期/时间。 如果请求在到达 LUIS 之前来自机器人或另一个集中式应用程序，则更正 LUIS 使用的时区。

### <a name="v3-prediction-api-to-alter-timezone"></a>用于更改时区的 V3 预测 API

在 V3 中，`datetimeReference` 确定时区偏移量。 请详细了解 [V3 预测](luis-migration-api-v3.md#v3-post-body)。

### <a name="v2-prediction-api-to-alter-timezone"></a>用于更改时区的 V2 预测 API
可以通过以下方式更正时区：根据 API 版本，使用 `timezoneOffset` 参数将用户的时区添加到终结点。 要更改时间，此参数值应为正数或负数（以分钟为单位）。

#### <a name="v2-prediction-daylight-savings-example"></a>V2 预测夏令时示例
如果需要返回的预生成 datetimeV2 来调整夏令时，则应对该[终结点](https://dev.cognitive.azure.cn/docs/services/5819c76f40a6350ce09de1ac/operations/5819c77140a63516d81aee78)查询使用值为正数/负数（以分钟为单位）的 querystring 参数。

增加 60 分钟：

`https://{region}.api.cognitive.azure.cn/luis/v2.0/apps/{appId}?q=Turn the lights on?timezoneOffset=60&verbose={boolean}&spellCheck={boolean}&staging={boolean}&bing-spell-check-subscription-key={string}&log={boolean}`

减去 60 分钟：

`https://{region}.api.cognitive.azure.cn/luis/v2.0/apps/{appId}?q=Turn the lights on?timezoneOffset=-60&verbose={boolean}&spellCheck={boolean}&staging={boolean}&bing-spell-check-subscription-key={string}&log={boolean}`

#### <a name="v2-prediction-c-code-determines-correct-value-of-parameter"></a>V2 预测 C# 代码确定正确的参数值

下面的 C# 代码使用 [TimeZoneInfo](https://docs.microsoft.com/dotnet/api/system.timezoneinfo) 类的 [FindSystemTimeZoneById](https://docs.microsoft.com/dotnet/api/system.timezoneinfo.findsystemtimezonebyid#examples) 方法基于系统时间来确定正确的偏移值：

```csharp
// Get CST zone id
TimeZoneInfo targetZone = TimeZoneInfo.FindSystemTimeZoneById("Central Standard Time");

// Get local machine's value of Now
DateTime utcDatetime = DateTime.UtcNow;

// Get Central Standard Time value of Now
DateTime cstDatetime = TimeZoneInfo.ConvertTimeFromUtc(utcDatetime, targetZone);

// Find timezoneOffset/datetimeReference
int offset = (int)((cstDatetime - utcDatetime).TotalMinutes);
```
