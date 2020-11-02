---
title: 使用 Kusto.Explorer
description: 了解如何使用 Kusto.Explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: conceptual
origin.date: 05/19/2020
ms.date: 09/30/2020
ms.openlocfilehash: 35c56439d0c44bcf4aa5aa27286b23a826cfdc46
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105306"
---
# <a name="using-kustoexplorer"></a>使用 Kusto.Explorer

Kusto.Explorer 是一个桌面应用程序，可让你在易于使用的用户界面中使用 Kusto 查询语言来浏览数据。 本文介绍如何使用搜索和查询模式、如何共享查询以及如何管理群集、数据库和表。

## <a name="search-mode"></a>Search++ 模式

使用 Search++ 模式，可在一个或多个表中使用搜索语法搜索字词。

1. 在“主页”选项卡的“查询”下拉列表中选择“Search++”。
1. 选择“多个表”。
1. 在“选择表”下，定义要搜索的表。
1. 在“编辑”框中，输入搜索短语，然后选择“开始”。
1. 表/时间段网格的热图显示出现的字词以及它们出现的位置。

    :::image type="content" source="images/kusto-explorer-using/search-plus-plus.png" alt-text="Search++ Kusto Explorer":::

1. 在网格中选择一个单元格，然后选择“查看详细信息”，在结果窗格中显示相关条目。

    :::image type="content" source="images/kusto-explorer-using/search-plus-plus-results.png" alt-text="Search++ Kusto Explorer":::

## <a name="query-mode"></a>查询模式

Kusto.Explorer 包含一个功能强大的脚本模式，可用于编写、编辑和运行即席查询。 此脚本模式附带语法突出显示和 IntelliSense，可方便你快速加深对 Kusto 查询语言的了解。

本部分介绍如何在 Kusto.Explorer 中运行基本查询，以及如何将参数添加到查询。

## <a name="basic-queries"></a>基本查询

如果你有表日志，可以开始浏览它们：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->

```kusto
StormEvents | count 
```

当光标位于此行上时，它的颜色为灰色。 按 F5 运行查询。 

下面提供了一些查询示例：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->

```kusto
// Take 10 lines from the table. Useful to get familiar with the data
StormEvents | limit 10 
```

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->

```kusto
// Filter by EventType == 'Flood' and State == 'California' (=~ means case insensitive) 
// and take sample of 10 lines
StormEvents 
| where EventType == 'Flood' and State =~ 'California'
| limit 10
```

:::image type="content" source="images/kusto-explorer-using/basic-query.png" alt-text="Search++ Kusto Explorer":::

详细了解 [Kusto 查询语言](https://docs.azure.cn/kusto/query/)。

> [!NOTE]
> 查询表达式中的空白行可能影响执行查询的那一部分。
>
> 如果未选定文本，则假定以空行分隔查询或命令。
> 如果选定了文本，则运行选定的文本。

## <a name="client-side-query-parameterization"></a>客户端查询参数化

> [!NOTE]
> Kusto 中有两种类型的查询参数化技术：
> * [语言集成查询参数化](../query/queryparametersstatement.md)作为查询引擎的一部分来实现，旨在供以编程方式查询服务的应用程序使用。 本文档不介绍此方法。
>
> * 如下所述，客户端查询参数化只是 Kusto.Explorer 应用程序的一项功能。 这相当于在将查询发送给服务执行之前对查询使用字符串替换操作。 下述语法不是查询语言本身的一部分，并且在通过 Kusto.Explorer 以外的方式将查询发送到服务时不能使用。

如果在多个查询或多个选项卡中使用相同的值，则在使用该值的每一处进行更改非常不方便。 这就是 Kusto.Explorer 支持查询参数的原因。 查询参数在选项卡之间共享，以便可以轻松重复使用。 参数由 {} 括号表示。 例如： `{parameter1}`

脚本编辑器突出显示查询参数：

:::image type="content" source="images/kusto-explorer-using/parametrized-query-1.png" alt-text="Search++ Kusto Explorer":::

可以轻松定义和编辑现有查询参数：


:::image type="content" source="images/kusto-explorer-using/parametrized-query-2.png" alt-text="Search++ Kusto Explorer":::


:::image type="content" source="images/kusto-explorer-using/parametrized-query-3.png" alt-text="Search++ Kusto Explorer":::

脚本编辑器还包含已定义的查询参数的 IntelliSense：

:::image type="content" source="images/kusto-explorer-using/parametrized-query-4.png" alt-text="Search++ Kusto Explorer":::

你可以具有多组参数（在“参数集”组合框中列出）。
选择“新增”或“删除现有”以操作参数集列表 。

:::image type="content" source="images/kusto-explorer-using/parametrized-query-5.png" alt-text="Search++ Kusto Explorer":::

## <a name="share-queries-and-results"></a>共享查询和结果

在 Kusto.Explorer 中，可以通过电子邮件共享查询和结果。 还可以创建将在浏览器中打开和运行查询的深层链接。

### <a name="share-queries-and-results-by-email"></a>通过电子邮件共享查询和结果

Kusto.Explorer 提供了一种便捷方式，可以通过电子邮件共享查询和查询结果。

1. 在 Kusto.Explorer 中[运行查询](#basic-queries)。
1. 在“主页”选项卡的“共享”部分，选择“导出到剪贴板”（或按 Ctrl+Shift+C）。

    :::image type="content" source="images/kusto-explorer-using/menu-export.png" alt-text="Search++ Kusto Explorer":::

    Kusto.Explorer 将以下内容粘贴到剪贴板：
     * 你的查询
     * 查询结果（表或图表）
     * Kusto 群集和数据库的连接详细信息
     * 将自动重新运行查询的链接

1. 将剪贴板中的内容粘贴到新的电子邮件中。

    :::image type="content" source="images/kusto-explorer-using/share-results-2.png" alt-text="Search++ Kusto Explorer":::

### <a name="deep-linking-queries"></a>深层链接查询

可以创建一个 URI，当在浏览器中打开该 URI 时，它将在本地打开 Kusto.Explorer，并在指定的 Kusto 数据库上运行特定的查询。

> [!NOTE] 
> 出于安全原因，禁用了控制命令的深层链接。

#### <a name="creating-a-deep-link"></a>创建深层链接

创建深层链接最简单的方法是在 Kusto.Explorer 中编写查询，然后使用 `Export to Clipboard` 将查询（包括深层链接和结果）复制到剪贴板。 然后，你可以通过电子邮件共享它。
        
复制到电子邮件时，深层链接以小字体显示。 例如：

https://help.kusto.chinacloudapi.cn:443/Samples [[单击以运行查询](https://help.kusto.chinacloudapi.cn/Samples?web=0&query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVuDlqlEoLs3NTSzKrEpVSM4vzSvR0FRIqlRIyszTCC5JLCoJycxN1VEwT9EEKS1KzUtJLVIoAYolZwAlFQCB3oo%2bTAAAAA%3d%3d)]

第一个链接打开 Kusto.Explorer 并相应地设置群集和数据库上下文。
第二个链接 (`Click to run query`) 是深层链接。 如果将链接移至电子邮件并按 Ctrl+K，则可以看到实际的 URL：

https://help.kusto.chinacloudapi.cn/Samples?web=0&query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVuDlqlEoLs3NTSzKrEpVSM4vzSvR0FRIqlRIyszTCC5JLCoJycxN1VEwT9EEKS1KzUtJLVIoAYolZwAlFQCB3oo%2bTAAAAA%3d%3d

#### <a name="deep-links-and-parametrized-queries"></a>深层链接和参数化查询

可以将参数化查询与深层链接结合使用。

1. 创建要构成为参数化查询的查询（如 `KustoLogs | where Timestamp > ago({Period}) | count`） 
1. 为 URI 中的每个查询参数提供一个参数，例如： 
    
    `https://<your_cluster>.kusto.chinacloudapi.cn/MyDatabase?
web=0&query=KustoLogs+%7c+where+Timestamp+>+ago({Period})+%7c+count&Period=1h`

    将 your_cluster 替换为 Azure 数据资源管理器群集的名称&lt;&gt;。

#### <a name="limitations"></a>限制

由于浏览器限制、HTTP 代理和验证链接的工具（如 Microsoft Outlook），查询的字符数不得超过 2000 个。 该限制是近似值，因为它取决于群集和数据库名称的长度。 有关详细信息，请参阅 [https://support.microsoft.com/kb/208427](https://support.microsoft.com/kb/208427)。 

要减少达到字符限制的几率，请参阅[获取更短的链接](#getting-shorter-links)。

URI 的格式为：`https://<ClusterCname>.kusto.chinacloudapi.cn/<DatabaseName>web=0?query=<QueryToExecute>`

例如：[https://help.kusto.chinacloudapi.cn/Samples?web=0query=StormEvents+%7c+limit+10](https://help.kusto.chinacloudapi.cn/Samples?web=0query=StormEvents+%7c+limit+10)
 
该 URI 将打开 Kusto.Explorer，连接到 `Help` Kusto 群集，并在 `Samples` 数据库上运行指定的查询。 如果已经有 Kusto.Explorer 实例在运行，则正在运行的实例将打开一个新选项卡并在其中运行查询。

### <a name="getting-shorter-links"></a>获取更短的链接

查询可能会变得很长。 要减少查询超过最大长度的几率，请使用 Kusto 客户端库中提供的 `String Kusto.Data.Common.CslCommandGenerator.EncodeQueryAsBase64Url(string query)` 方法。 此方法可生成更精简的查询版本。 Kusto.Explorer 也可以识别较短的格式。

https://help.kusto.chinacloudapi.cn/Samples?web=0&query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVuDlqlEoLs3NTSzKrEpVSM4vzSvR0FRIqlRIyszTCC5JLCoJycxN1VEwT9EEKS1KzUtJLVIoAYolZwAlFQCB3oo%2bTAAAAA%3d%3d

通过应用下一个转换，使查询更精简：

```csharp
 UrlEncode(Base64Encode(GZip(original query)))
```

## <a name="kustoexplorer-command-line-arguments"></a>Kusto.Explorer 命令行参数

使用命令行参数来配置工具以便在启动时执行其他功能。 例如，加载脚本并连接到群集。 因此，命令行参数不能替代任何 Kusto.Explorer 功能。

命令行参数作为用于打开应用程序的 URL 的一部分传递，类似于[查询深层链接](#creating-a-deep-link)。

### <a name="command-line-argument-syntax"></a>命令行参数语法

Kusto.Explorer 支持以下语法中的几个命令行参数（顺序很重要）：

[ *LocalScriptFile* ] [ *QueryString* ]

* LocalScriptFile 是本地计算机上脚本文件的名称，该文件必须具有扩展名 `.kql`。 如果此文件存在，则 Kusto.Explorer 会在启动时自动加载此文件。
* QueryString 是使用 HTTP 查询字符串格式的字符串。 该方法提供了其他属性，如下表所述。

例如，要使用名为 `c:\temp\script.kql` 的脚本文件启动 Kusto.Explorer，并将其配置为与群集 `help`、数据库 `Samples` 通信，请使用以下命令：

```kusto
Kusto.Explorer.exe c:\temp\script.kql uri=https://help.kusto.chinacloudapi.cn/Samples;Fed=true&name=Samples
```

|参数  |说明                                                               |
|----------|--------------------------------------------------------------------------|
|**要执行的查询**                                                                 |
|`query`   |要执行的查询（base64 编码）。 如果为空，则使用 `querysrc`。          |
|`querysrc`|保存要执行的查询的文件或 blob 的 URL（如果 `query` 为空）。|
|**与 Kusto 群集的连接**                                                  |
|`uri`     |要连接到的 Kusto 群集的连接字符串。                 |
|`name`    |与 Kusto 群集的连接的显示名称。                  |
|**连接组**                                                                 |
|`path`    |要下载的连接组文件的 URL（URL 编码）。             |
|`group`   |连接组的名称。                                         |
|`filename`|保存连接组的本地文件。                              |


## <a name="manage-clusters-databases-tables-or-function-authorized-principals"></a>管理群集、数据库、表或函数授权主体

> [!NOTE]
> 只有[管理员](../management/access-control/role-based-authorization.md)才能在自己的范围中添加或删除授权主体。

在[连接面板](kusto-explorer.md#connections-tab)中右键单击目标实体，然后选择“管理群集授权主体”。 （也可以从“管理”菜单中选择此选项。）

:::image type="content" source="images/kusto-explorer-using/right-click-manage-authorized-principals.png" alt-text="Search++ Kusto Explorer":::

:::image type="content" source="images/kusto-explorer-using/manage-authorized-principals-window.png" alt-text="Search++ Kusto Explorer":::

* 要添加新的授权主体，请选择“添加主体”，提供主体详细信息，然后确认操作。
    
    :::image type="content" source="images/kusto-explorer-using/add-authorized-principals-window.png" alt-text="Search++ Kusto Explorer":::

    :::image type="content" source="images/kusto-explorer-using/confirm-add-authorized-principals.png" alt-text="Search++ Kusto Explorer":::

* 要删除现有授权主体，请选择“删除主体”并确认操作。

    :::image type="content" source="images/kusto-explorer-using/confirm-drop-authorized-principals.png" alt-text="Search++ Kusto Explorer":::


## <a name="next-steps"></a>后续步骤

* [Kusto.Explorer 键盘快捷方式](kusto-explorer-shortcuts.md)
* [Kusto.Explorer 选项](kusto-explorer-options.md)
* [Kusto.Explorer 故障排除](kusto-explorer-troubleshooting.md)

详细了解 Kusto.Explorer 工具和实用程序：
* [Kusto.Explorer 代码分析器](kusto-explorer-code-analyzer.md)
* [Kusto.Explorer 代码导航](kusto-explorer-codenav.md)
* [Kusto.Explorer 代码重构](kusto-explorer-refactor.md)
* [Kusto 查询语言 (KQL)](https://docs.azure.cn/kusto/query/)
