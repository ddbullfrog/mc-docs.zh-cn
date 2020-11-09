---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/04/2020
title: 小组件 - Azure Databricks
description: 了解如何使用输入小组件，以便将参数添加到笔记本和仪表板。
ms.openlocfilehash: 6c22979559150c53ad98e7b6f539c96e2ec7bbea
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106796"
---
# <a name="widgets"></a>小组件

使用输入小组件，可将参数添加到笔记本和仪表板。 小组件 API 包含多个调用，可用于创建各种类型的输入小组件、删除小组件以及获取绑定值。

小组件最适用于：

* 构建可使用不同参数重新执行的笔记本或仪表板
* 使用不同参数快速浏览单个查询的结果

> [!TIP]
>
> 通过以下命令查看有关 Scala、Python 和 R 中的小组件 API 的文档：

```scala
dbutils.widgets.help()
```

## <a name="widget-types"></a>小组件类型

有 4 种类型的小组件：

* `text`：在文本框中输入一个值。
* `dropdown`：从提供的值列表中选择一个值。
* `combobox`：文本和下拉列表的组合。 从提供的列表中选择一个值，或在文本框中输入一个值。
* `multiselect`：从提供的值列表中选择一个或多个值。

小组件下拉列表和文本框会立即显示在笔记本工具栏后面。

> [!div class="mx-imgBorder"]
> ![小组件下拉列表](../_static/images/widgets/widget-dropdown.png)

## <a name="widget-api"></a>小组件 API

小组件 API 设计为在 Scala、Python 和 R 中保持一致。SQL 中的小组件 API 略有不同，但与其他语言一样强大。 可通过 [Databricks 实用程序](../dev-tools/databricks-utils.md)界面来管理小组件。

```python
dbutils.widgets.dropdown("X123", "1", [str(x) for x in range(1, 10)])

dbutils.widgets.dropdown("1", "1", [str(x) for x in range(1, 10)], "hello this is a widget")

dbutils.widgets.dropdown("x123123", "1", [str(x) for x in range(1, 10)], "hello this is a widget")

dbutils.widgets.dropdown("x1232133123", "1", [str(x) for x in range(1, 10)], "hello this is a widget 2")
```

### <a name="widget-example"></a>小组件示例

创建一个简单的下拉小组件。

```python
dbutils.widgets.dropdown("X", "1", [str(x) for x in range(1, 10)])
```

与小组件面板中的小组件进行交互。

> [!div class="mx-imgBorder"]
> ![小组件下拉列表](../_static/images/widgets/widget-dropdown2.png)

可通过调用访问小组件的当前值：

```python
dbutils.widgets.get("X")
```

最后，可删除笔记本中的某个小组件或所有小组件：

```python
dbutils.widgets.remove("X")

dbutils.widgets.removeAll()
```

> [!IMPORTANT]
>
> 如果添加了命令来删除小组件，则之后不能在同一单元格中再添加命令来创建小组件。 必须在另一个单元格中创建小组件。

### <a name="widgets-in-scala-python-and-r"></a>Scala、Python 和 R 中的小组件

若要查看每种方法的详细 API 文档，请使用 `dbutils.widgets.help("<method-name>")`。 帮助 API 在所有语言中都是相同的。 例如：

```python
dbutils.widgets.help("dropdown")
```

可通过传递唯一标识名称、默认值和默认选项列表以及可选标签来创建下拉小组件。 创建后，下拉输入小组件会立即出现在笔记本顶部。 这些输入小组件是笔记本级别的实体。

如果尝试创建已存在的小组件，现有小组件的配置将会被新选项覆盖。

### <a name="widgets-in-sql"></a>SQL 中的小组件

在 SQL 中创建小组件的 API 略有不同，但与其他语言的 API 一样强大。 下面是创建文本输入小组件的示例。

```sql
CREATE WIDGET TEXT y DEFAULT "10"
```

若要在 SQL 的下拉小组件中指定可选值，你可编写一个子查询。 子查询生成的表的第一列将确定这些值。

以下单元格从表的子查询创建一个下拉小组件。

```sql
CREATE WIDGET DROPDOWN cuts DEFAULT "Good" CHOICES SELECT DISTINCT cut FROM diamonds
```

创建下拉小组件时指定的默认值必须为可选值之一，并且必须指定为字符串文本。 若要访问 SQL 中的输入小组件的当前选定值，可在查询中使用特殊 UDF 函数。 函数是 `getArgument()`。 例如：

```sql
SELECT COUNT(*) AS numChoices, getArgument("cuts") AS cuts FROM diamonds WHERE cut = getArgument("cuts")
```

> [!NOTE]
>
> `getArgument` 作为 Scala UDF 实现，在启用了表 ACL 的高并发群集上不受支持。 在此类群集上，可使用 [SQL 中的旧版输入小组件](#legacy-input)中所示的语法。

最后，你可使用 SQL 命令删除小组件：

```sql
REMOVE WIDGET cuts
```

> [!IMPORTANT]
>
> 通常情况下，不能使用小组件在笔记本中的不同语言之间传递参数。 可在 Python 单元格中创建小组件 `arg1`，如果是逐个单元格运行，还可在 SQL 或 Scala 单元格中使用它。 但是，如果使用“Run All”执行所有命令或将笔记本作为作业运行，则无法正常工作。 为了解决这一限制，我们建议你为每种语言创建一个笔记本，并在[运行笔记本](notebook-workflows.md)时传递参数。

#### <a name="legacy-input-widgets-in-sql"></a><a id="legacy-input"> </a><a id="legacy-input-widgets-in-sql"> </a>SQL 中的旧版输入小组件

使用 `$<parameter>` 语法在 SQL 查询中创建小组件的旧方法仍和以前一样有用。 以下是示例：

```sql
SELECT * FROM diamonds WHERE cut LIKE '%$cuts%'
```

> [!NOTE]
>
> 如果要编写 SQL 查询，无论是在 SQL 笔记本中编写还是使用不同的默认语言在笔记本的 `%sql` [magic 命令](notebooks-use.md#language-magic)中编写，均不能在标识符中使用 `$`，因为它会被解释为参数。 若要在 SQL 命令单元中转义 `$`，请使用 `$\`。 例如，若要定义标识符 `$foo`，请将其编写为 `$\foo`。

## <a name="configure-widget-settings"></a>配置小组件设置

你可配置在选择新值时小组件的行为，以及小组件面板是否始终固定到笔记本顶部。

1. 单击 ![gear](../_static/images/widgets/gear.png) 图标，其位于“小组件”面板的右端。
2. 在弹出式“小组件面板设置”对话框中，选择小组件的执行行为。

   > [!div class="mx-imgBorder"]
   > ![小组件设置](../_static/images/widgets/widget-settings.png)

   * **运行笔记本** ：每次选择新值时，整个笔记本都会重新运行。
   * **运行已访问的命令** ：每次选择新值时，只有检索该特定小组件值的单元格会重新运行。 这是创建小组件时的默认设置。

     > [!NOTE]
     >
     > SQL 单元格不会在此配置中重新运行。

   * **不执行任何操作** ：每次选择新值时，任何内容都不会重新运行。

### <a name="notebook"></a>笔记本

可在以下笔记本中看到“运行已访问的命令”设置如何工作的演示。 `year` 小组件是使用 `2014` 设置创建的，用于数据帧 API 和 SQL 命令。

> [!div class="mx-imgBorder"]
> ![小组件](../_static/images/widgets/widget-demo.png)

将 `year` 小组件的设置更改为 `2007` 时，DataFrame 命令将重新运行，但 SQL 命令不会重新运行。

#### <a name="widget-demo-notebook"></a>小组件演示笔记本

[获取笔记本](../_static/notebooks/widget-demo.html)

## <a name="widgets-in-dashboards"></a>仪表板中的小组件

从包含输入小组件的笔记本创建仪表板时，所有小组件都将显示在仪表板顶部。 在演示模式下，每次更新小组件的值时，都可单击“更新”按钮来重新运行笔记本并使用新值更新仪表板。

> [!div class="mx-imgBorder"]
> ![小组件的仪表板](../_static/images/widgets/widget-dashboard2.png)

## <a name="use-widgets-with-run"></a>将小组件与 %run 配合使用

如果[运行包含小组件的笔记本](notebooks-use.md#run)，指定的笔记本会使用该小组件的默认值运行。 还可向小组件传入值。 例如：

```bash
%run /path/to/notebook $X="10" $Y="1"
```

此示例将运行指定的笔记本，并将 `10` 传递给小组件 X，将 `1` 传递给小组件 Y。