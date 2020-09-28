---
title: Kusto.Explorer 安装和用户界面
description: 了解 Kusto.Explorer 的功能以及如何借助 Kusto.Explorer 来浏览数据
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: conceptual
origin.date: 05/19/2020
ms.date: 09/24/2020
ms.openlocfilehash: 9c22d8214226f4f93b3450ffec6f2cdf3bf806ec
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146378"
---
# <a name="kustoexplorer-installation-and-user-interface"></a>Kusto.Explorer 安装和用户界面

Kusto.Explorer 是一个富桌面应用程序，可让你在易于使用的用户界面中使用 Kusto 查询语言来浏览数据。 此概述介绍了如何开始设置 Kusto.Explorer，并说明了将使用的用户界面。

借助 Kusto.Explorer，你可以：
* [查询数据](kusto-explorer-using.md#query-mode)。
* [跨表搜索数据](kusto-explorer-using.md#search-mode)。
* [可视化各种图形中的数据](#visualizations-section)。
* 通过电子邮件或使用深度链接[共享查询和结果](kusto-explorer-using.md#share-queries-and-results)。

## <a name="installing-kustoexplorer"></a>安装 Kusto.Explorer

* 从 [https://aka.ms/ke](https://aka.ms/ke) 下载并安装安装 Kusto.Explorer 工具

* 改为使用浏览器 (`https://<your_cluster>.kusto.chinacloudapi.cn.`) 访问 Kusto 群集
   将 your_cluster 替换为 Azure 数据资源管理器群集的名称&lt;&gt;。

### <a name="using-chrome-and-kustoexplorer"></a>使用 Chrome 和 Kusto.Explorer

如果使用 Chrome 作为默认浏览器，请确保安装适用于 Chrome 的 ClickOnce 扩展：

[https://chrome.google.com/webstore/detail/clickonce-for-google-chro/kekahkplibinaibelipdcikofmedafmb/related?hl=en-US](https://chrome.google.com/webstore/detail/clickonce-for-google-chro/kekahkplibinaibelipdcikofmedafmb/related?hl=en-US)

## <a name="overview-of-the-user-interface"></a>用户界面概述

Kusto.Explorer 用户界面的设计使用基于选项卡和面板的布局，这类似于其他 Microsoft 产品的设计： 

1. 在[菜单面板](#menu-panel)上浏览选项卡以执行各种操作
2. 在[连接面板](#connections-panel)中管理连接
3. 在脚本面板中创建要运行的脚本
4. 在结果面板中查看脚本的结果

:::image type="content" source="images/kusto-explorer/ke-start.png" alt-text="Kusto Explorer 开始页面":::

## <a name="menu-panel"></a>菜单面板

Kusto.Explorer 菜单面板包含以下选项卡：

* [主页](#home-tab)
* [文件](#file-tab)
* [连接](#connections-tab)
* [视图](#view-tab)
* [工具](#tools-tab)
* [监视](#monitoring-tab)
* [Management](#management-tab)
* [帮助](#help-tab)

### <a name="home-tab"></a>“主文件夹”选项卡

:::image type="content" source="images/kusto-explorer/home-tab.png" alt-text="Kusto Explorer“主页”选项卡":::

“主页”选项卡显示最近使用过的功能，分为以下几个部分：

* [查询](#query-section)
* [共享](#share-section)
* [可视化效果](#visualizations-section)
* [视图](#view-section)
* [帮助](#help-tab) 

### <a name="query-section"></a>“查询”部分

:::image type="content" source="images/kusto-explorer/home-query-menu.png" alt-text="Kusto Explorer“查询”部分":::

|菜单|    行为|
|----|----------|
|模式下拉菜单 | <ul><li>查询模式：将查询窗口切换为[脚本模式](kusto-explorer-using.md#query-mode)。 可以加载命令并将其另存为脚本（默认值）</li> <li> 搜索模式：单个查询模式，其中输入的每个命令都将立即被处理，并在结果窗口中显示结果</li> <li>Search++ 模式：可在一个或多个表中使用搜索语法搜索字词。 了解有关使用 [Search++ 模式](kusto-explorer-using.md#search-mode)的详细信息</li></ul> |
|新选项卡| 打开新选项卡以查询 Kusto |

### <a name="share-section"></a>“共享”部分

:::image type="content" source="images/kusto-explorer/home-share-menu.png" alt-text="Kusto Explorer 共享菜单":::

|菜单|    行为|
|----|----------|
|将数据导出到剪贴板|    将查询和数据集导出到剪贴板。 如果显示图表，则会将图表导出为位图| 
|将结果导出到剪贴板| 将数据集导出到剪贴板。 如果显示图表，则会将图表导出为位图| 
|将查询导出到剪贴板| 将查询导出到剪贴板|

### <a name="visualizations-section"></a>“可视化效果”部分

:::image type="content" source="images/kusto-explorer/home-visualizations-menu.png" alt-text="Kusto Explorer 菜单可视化效果":::

|菜单         | 行为|
|-------------|---------|
|面积图      | 显示一个面积图，其中 X 轴是第一列（必须是数字）。 所有数字列都映射到不同的序列（Y 轴） |
|柱形图 | 显示一个柱形图，其中所有数值列都映射到不同的序列（Y 轴）。 数值前的文本列是 X 轴（可以在 UI 中进行控制）|
|条形图    | 显示一个条形图，其中所有数值列都映射到不同的序列（X 轴）。 数值前的文本列是 Y 轴（可以在 UI 中进行控制）|
|堆积面积图      | 显示一个堆积面积图，其中 X 轴是第一列（必须是数字）。 所有数字列都映射到不同的序列（Y 轴） |
|时间线图表   | 显示一个时间图，其中 X 轴是第一列（必须是日期/时间）。 所有数字列都映射到不同的序列（Y 轴）。|
|折线图   | 显示一个折线图，其中 X 轴是第一列（必须是数字）。 所有数字列都映射到不同的序列（Y 轴）。|
|[异常图](#anomaly-chart)|    类似于时间图，但使用机器学习异常算法查找时序数据中的异常。 Kusto.Explorer 使用 [series_decompose_anomalies](../query/series-decompose-anomaliesfunction.md) 函数来检测异常情况。
|饼图    |    显示一个饼图，其中色轴是第一列。 θ 轴（必须是一个度量值，转换为百分比）是第二列。|
|时间阶梯 |    显示一个梯形图，其中 X 轴是最后两列（必须是日期/时间）。 Y 轴是其他列的组合。|
|散点图| 显示一个散点图，其中 X 轴是第一列（必须是数字）。 所有数字列都映射到不同的序列（Y 轴）。|
|透视图  | 显示一个透视表和透视图，选择数据、列、行和各种图表类型时具有全面的灵活性。| 
|时间透视   | 在事件时间线上进行交互式导航（沿时间轴旋转）|

> [!NOTE]
> <a id="anomaly-chart">异常情况图表</a>：该算法需要时序数据，其中包含以下两列：
>* 固定时间间隔 bucket 中的时间
>* 异常情况检测的数值 若要在 Kusto.Explorer 中生成时序数据，请按时间字段汇总并制定，按时间字段汇总并指定时间 bucket 箱。

### <a name="view-section"></a>“视图”部分

:::image type="content" source="images/kusto-explorer/home-view-menu.png" alt-text="Kusto Explorer“视图”菜单":::

|菜单           | 行为|
|---------------|---------|
|完整视图模式 | 通过隐藏功能区菜单和连接面板来最大化工作空间。 可通过选择“主页” > “完整视图模式”或按 F11 退出完整视图模式  。|
|隐藏空列| 从数据网格中删除空列|
|折叠单数列| 折叠包含单数值的列|
|浏览列值| 显示列值分布|
|增大字体  | 增大“查询”选项卡和“结果”数据网格的字体大小|  
|减小字体  | 减小“查询”选项卡和“结果”数据网格的字体大小|

>[!NOTE]
> 数据视图设置：
>
> Kusto.Explorer 按每组唯一列跟踪使用的设置。 当对列进行重新排序或删除列时，将保存数据视图，并在检索到具有相同列的数据时重用该视图。 若要将该视图重置为其默认值，请在“视图”选项卡中，选择“重置视图” 。 

## <a name="file-tab"></a>“文件”选项卡

:::image type="content" source="images/kusto-explorer/file-tab.png" alt-text="Kusto Explorer“文件”选项卡":::

|菜单| 行为|
|---------------|---------|
||---------查询脚本---------|
|新选项卡 | 打开新选项卡窗口以查询 Kusto |
|打开文件| 将 *.kql 文件中的数据加载到活动脚本面板中|
|保存到文件| 将活动脚本面板的内容加载到 *.kql 文件中|
|关闭选项卡| 关闭当前选项卡窗口|
||---------保存数据---------|
|将数据导出到 CSV       | 将数据导出到 CSV（逗号分隔值）文件| 
|将数据导出到 JSON      | 将数据导出到 JSON 格式的文件|
|将数据导出到 Excel     | 将数据导出到 XLSX (Excel) 文件|
|将数据导出到文本      | 将数据导出到 TXT（文本）文件| 
|将数据导出到 KQL 脚本| 将查询导出到脚本文件| 
|将数据导出到结果   | 将查询和数据导出到结果 (QRES) 文件|
|运行查询并保存到 CSV |运行查询并将结果保存到本地 CSV 文件|
||---------加载数据---------|
|从结果|    从结果 (QRES) 文件加载查询和数据| 
||---------剪贴板---------|
|将查询和结果导出到剪贴板|    将查询和数据集导出到剪贴板。 如果显示图表，则会将图表导出为位图| 
|将结果导出到剪贴板| 将数据集导出到剪贴板。 如果显示图表，则会将图表导出为位图| 
|将查询导出到剪贴板| 将查询导出到剪贴板|
||---------结果---------|
|清除结果缓存| 清除以前执行的查询的缓存结果| 

## <a name="connections-tab"></a>“连接”选项卡

:::image type="content" source="images/kusto-explorer/connections-tab.png" alt-text="Kusto Explorer“连接”选项卡":::

|菜单|行为|
|----|----------|
||---------组---------|
|添加组| 添加新的 Kusto 服务器组|
|重命名组| 重命名现有的 Kusto 服务器组|
|删除组| 删除现有 Kusto 服务器组|
||---------群集---------|
|导入连接| 从指定连接的文件中导入连接|
|导出连接| 将连接导出到文件|
|添加连接| 添加新的 Kusto 服务器连接| 
|编辑连接| 打开一个对话框以编辑 Kusto 服务器连接属性|
|删除连接| 删除到 Kusto 服务器的现有连接|
|刷新| 刷新 Kusto 服务器连接的属性|
||---------*安全性*---------|
|检查 ADD 主体| 显示当前活动用户详细信息|
|从 AAD 注销| 注销连接到 AAD 的当前用户|
||---------数据范围---------|
|缓存范围|<ul><li>仅对[热数据缓存](../management/cachepolicy.md)执行热 DataExecute 查询</li><li>所有数据：对所有可用数据执行查询（默认）</li></ul> |
|日期/时间列| 用于时间初级筛选器的列的名称|
|时间筛选器| 时间初级筛选器的值|

## <a name="view-tab"></a>“视图”选项卡

:::image type="content" source="images/kusto-explorer/view-tab.png" alt-text="Kusto Explorer“视图”选项卡":::

|菜单|行为|
|----|----------|
||---------外观---------|
|完整视图模式 | 通过隐藏功能区菜单和连接面板来最大化工作空间|
|增大字体  | 增大“查询”选项卡和“结果”数据网格的字体大小|  
|减小字体  | 减小“查询”选项卡和“结果”数据网格的字体大小|
|重置布局|重置工具的停靠控件和窗口的布局|
|重命名“文档”选项卡 |重命名所选的选项卡 |
||---------数据视图---------|
|重置视图| 将[数据视图设置](#dvs)重置为其默认值 |
|浏览列值|显示列值分布|
|将焦点置于查询统计信息|查询完成时，将焦点更改为查询统计信息，而不是查询结果|
|隐藏重复项|切换从查询结果中删除重复行|
|隐藏空列|切换从查询结果中删除空列|
|折叠单数列|切换折叠包含单数值的列|
||---------数据筛选---------|
|筛选搜索中的行|切换选项以仅显示查询结果搜索中的匹配行 (Ctrl+F)|
||---------可视化效果---------|
|可视化效果|请参阅上述的[可视化效果](#visualizations-section)。 |

> [!NOTE]
> <a id="dvs">数据视图设置：</a> 
>
> Kusto.Explorer 按每组唯一列跟踪使用的设置。 当对列进行重新排序或删除列时，将保存数据视图，并在检索到具有相同列的数据时重用该视图。 若要将该视图重置为其默认值，请在“视图”选项卡中，选择“重置视图” 。 

## <a name="tools-tab"></a>“工具”选项卡

:::image type="content" source="images/kusto-explorer/tools-tab.png" alt-text="Kusto Explorer“工具”选项卡":::

|菜单|行为|
|----|----------|
||---------*IntelliSense*---------|
|启用 IntelliSense| 启用和禁用脚本面板上的 IntelliSense|
||---------分析---------|
|查询分析器| 启动查询分析器工具|
|查询检查器 | 分析当前查询并输出一组适用的改进建议|
|计算器| 启动计算器|
||---------*分析*---------|
|分析报表| 打开包含多个预建报表的仪表板以进行数据分析|
||---------转换---------|
|查询到 Power BI| 将查询转换为适用于在 Power BI 中使用的格式|
||---------选项---------|
|重置选项| 将应用程序设置设置为默认值|
|选项| 打开用于配置应用程序设置的工具。 详细了解 [Kusto.Explorer 选项](kusto-explorer-options.md)。|

## <a name="monitoring-tab"></a>“监视”选项卡

:::image type="content" source="images/kusto-explorer/monitoring-tab.png" alt-text="Kusto Explorer“监视”选项卡":::

|菜单             | 行为|
|-----------------|---------| 
||---------监视---------|
|群集诊断 | 显示连接面板中当前选定的服务器组的运行状况摘要 | 
|最新数据：所有表| 显示当前所选数据库的所有表中的最新数据的摘要|
|最新数据：所选表|在状态栏中显示所选表中的最新数据| 

## <a name="management-tab"></a>“管理”选项卡

:::image type="content" source="images/kusto-explorer/management-tab.png" alt-text="Kusto Explorer“管理”选项卡":::

|菜单             | 行为|
|-----------------|---------|
||---------授权主体---------|
|管理群集授权主体 |支持为授权用户管理群集的主体| 
|管理数据库授权主体 | 支持为授权用户管理数据库的主体| 
|管理表授权主体 | 支持为授权用户管理表的主体| 
|管理函数授权主体 | 支持为授权用户管理函数的主体| 

## <a name="help-tab"></a>“帮助”选项卡

:::image type="content" source="images/kusto-explorer/help-tab.png" alt-text="Kusto Explorer“帮助”选项卡":::

|菜单             | 行为|
|-----------------|---------|
||---------文档---------|
|帮助             | 打开指向 Kusto 联机文档的链接  | 
|新增功能       | 打开列出所有 Kusto.Explorer 更改的文档|
|报告问题      |打开包含以下两个选项的对话框： <ul><li>报告与服务相关的问题</li><li>报告客户端应用程序中的问题</li></ul> | 
|建议功能  | 打开指向 Kusto 反馈论坛的链接 | 
|检查更新     | 检查 Kusto.Explorer 的版本是否有更新 | 

## <a name="connections-panel"></a>“连接”面板

:::image type="content" source="images/kusto-explorer/connections-panel.png" alt-text="Kusto Explorer“连接”面板":::

“连接”窗格显示所有已配置的群集连接。 对于每个群集，将显示它们存储的数据库、表和属性（列）。 选择项（在主面板中为搜索/查询设置隐式上下文），或双击项可将名称复制到“搜索/查询”面板。

如果实际架构很大（如具有数百个表的数据库），则可以通过按CTRL + F 并输入所需实体名称的子字符串（不区分大小写）来进行搜索。

Kusto.Explorer 支持从“查询”窗口控制“连接”面板，这对于脚本非常有用。 例如，你可以使用命令启动脚本文件，该命令通过使用以下语法指导 Kusto.Explorer 连接正通过脚本查询其数据的群集/数据库：

<!-- csl -->
```kusto
#connect cluster('help').database('Samples')

StormEvents | count
```

使用 `F5` 或类似内容运行每行。

### <a name="control-the-user-identity-connecting-to-kustoexplorer"></a>控制连接到 Kusto.Explorer 的用户标识

新连接的默认安全模式是 AAD 联合安全。 身份验证是通过 Azure Active Directory 使用默认 AAD 用户体验来完成的。

如果需要对身份验证参数进行更精细的控制，可以展开“高级:连接字符串”编辑框，并提供有效的 [Kusto 连接字符串](../api/connection-strings/kusto.md)值。

例如，存在于多个 AAD 租户中的用户有时需要使用其标识的特定投影。 为此，请提供一个连接字符串，例如下面的连接字符串（将大写的单词替换为特定值）：

```kusto
Data Source=https://CLUSTER_NAME.kusto.chinacloudapi.cn;Initial Catalog=DATABASE_NAME;AAD Federated Security=True;Authority Id=AAD_TENANT_OF_CLUSTER;User=USER_DOMAIN
```

* `AAD_TENANT_OF_CLUSTER` 是托管群集的 AAD 租户的域名或 AAD 租户 ID (GUID)。 这通常是拥有群集的组织的域名，例如 `contoso.com`。 
* USER_DOMAIN 是受邀加入该租户的用户的标识（例如 `user@example.com`）。 

>[!NOTE]
> 用户的域名不一定与托管群集的租户的域名相同。

:::image type="content" source="images/kusto-explorer/advanced-connection-string.png" alt-text="Kusto Explorer 高级连接字符串":::

## <a name="keyboard-shortcuts"></a>键盘快捷方式

你可能会发现，使用键盘快捷方式比使用鼠标执行操作的速度更快。 请查看此 [Kusto.Explorer 键盘快捷方式列表](kusto-explorer-shortcuts.md) 以了解详细信息。

## <a name="table-row-colors"></a>表行颜色

Kusto.Explorer 尝试在结果面板中解释每行的严重性或详细级别，并对它们进行相应的着色。 它通过将每列的非重复值与一组已知模式（“警告”、“错误”等）进行匹配来实现此目的。

若要修改输出配色方案或禁用此行为，请从“工具”菜单中，选择“选项” > “结果查看器” > “详细配色方案”   。

:::image type="content" source="images/kusto-explorer/ke-color-scheme.png" alt-text="Kusto Explorer 配色方案修改":::

## <a name="next-steps"></a>后续步骤

详细了解如何使用 Kusto.Explorer：

* [使用 Kusto.Explorer](kusto-explorer-using.md)
* [Kusto.Explorer 键盘快捷方式](kusto-explorer-shortcuts.md)
* [Kusto.Explorer 选项](kusto-explorer-options.md)
* [Kusto.Explorer 故障排除](kusto-explorer-troubleshooting.md)

详细了解 Kusto.Explorer 工具和实用程序：
* [Kusto.Explorer 代码分析器](kusto-explorer-code-analyzer.md)
* [Kusto.Explorer 代码导航](kusto-explorer-codenav.md)
* [Kusto.Explorer 代码重构](kusto-explorer-refactor.md)
* [Kusto 查询语言 (KQL)](/kusto/query/)
