---
title: 用户分析 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的用户分析。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 11/17/2019
ms.date: 09/30/2020
ms.openlocfilehash: 9bb3f3aa0f148d01c081054c39a168a4ad2bcfcb
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105075"
---
# <a name="user-analytics-plugins"></a>用户分析插件

本部分介绍用户分析方案的 Kusto 扩展（插件）。

|方案|插件|详细信息|用户体验|
|--------|------|--------|-------|
| 随着时间的推移对新用户计数 | [activity_counts_metrics](activity-counts-metrics-plugin.md)|返回每个时间范围的计数/dcounts/新计数。 会将每个时间范围与所有以前的时间范围进行比较|Kusto.Explorer：报表库|
| 给定时间段的同比：保留率/改动率和新用户 | [activity_metrics](activity-metrics-plugin.md)|返回每个时间范围的 `dcount`、保留率/改动率。 每个时间范围与之前的时间范围进行比较|Kusto.Explorer：报表库|
| 通过滑动窗口返回用户计数和 `dcount` | [sliding_window_counts](sliding-window-counts-plugin.md)|对于每个时间范围，将返回回溯期间（以滑动窗口的方式）的计数和 `dcount`|
| 新用户队列：保留率/改动率和新用户 | [new_activity_metrics](new-activity-metrics-plugin.md)|在新用户（在时间范围中首次出现的所有用户）队列之间进行比较。 每个队列将与以前的所有队列进行比较。 比较会考虑所有以前的时间范围|Kusto.Explorer：报表库|
|活动用户：非重复用户计数 |[active_users_count](active-users-count-plugin.md)|返回每个时间范围的非重复用户。 仅当用户在指定的回溯周期中出现在至少 X 个不同的周期中时，才考虑该用户。|
|用户参与度：DAU/WAU/MAU|[activity_engagement](activity-engagement-plugin.md)|比较某个内围时间范围（例如每日）和某个外围时间范围（例如每周），以计算参与度（例如 DAU/WAU）|Kusto.Explorer：报表库|
|会话：统计活动会话数|[session_count](session-count-plugin.md)|对会话进行计数，这里的会话由时间段定义 - 一个用户记录视为一个新会话（如果它在回溯期间尚未在当前记录中出现）|
||||
|漏斗图：上一个和下一个状态序列分析 | [funnel_sequence](funnel-sequence-plugin.md)|对已执行一系列事件的非重复用户，以及该系列事件执行前的事件或执行后的事件进行计数。 可用于构造 Sankey 关系图||
|漏斗图：序列完成分析|[funnel_sequence_completion](funnel-sequence-completion-plugin.md)|计算每个时间范围内完成指定序列的用户的非重复计数|
||||