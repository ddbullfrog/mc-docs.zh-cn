---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/13/2020
title: 使用笔记本 - Azure Databricks
description: 了解如何通过开发和运行单元格来使用笔记本。
ms.openlocfilehash: 9303406ae4b95e04e7d208740cefe70b93cab2e5
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106650"
---
# <a name="use-notebooks"></a>使用笔记本

笔记本是可运行的单元格（命令）的集合。 使用笔记本时，你主要是在开发和运行单元格。

UI 操作支持所有笔记本任务，但你也可以使用键盘快捷方式执行许多任务。 通过单击 ![“键盘”图标](../_static/images/notebooks/keyboard.png) 图标或选择“? > 快捷方式”来切换快捷方式显示。

> [!div class="mx-imgBorder"]
> ![键盘快捷键](../_static/images/notebooks/short-cuts.png)

## <a name="develop-notebooks"></a><a id="develop-notebooks"> </a><a id="notebook-develop"> </a>开发笔记本

本部分介绍如何开发笔记本单元格并在笔记本中导航。

### <a name="in-this-section"></a>本节内容：

* [关于笔记本](#about-notebooks)
* [添加单元格](#add-a-cell)
* [删除单元格](#delete-a-cell)
* [剪切单元格](#cut-a-cell)
* [选择多个单元格或所有单元格](#select-multiple-cells-or-all-cells)
* [默认语言](#default-language)
* [混合语言](#mix-languages)
* [包括文档](#include-documentation)
* [命令注释](#command-comments)
* [更改单元格显示](#change-cell-display)
* [显示行号和命令号](#show-line-and-command-numbers)
* [查找和替换文本](#find-and-replace-text)
* [自动完成](#autocomplete)
* [设置 SQL 格式](#format-sql)

### <a name="about-notebooks"></a>关于笔记本

笔记本包含一个工具栏，用于在笔记本中管理笔记本和执行操作：

> [!div class="mx-imgBorder"]
> ![笔记本工具栏](../_static/images/notebooks/toolbar.png)

此外还有一个或多个单元格（或命令），可以用来运行：

> [!div class="mx-imgBorder"]
> ![笔记本单元格](../_static/images/notebooks/cmd.png)

在单元格的最右侧，单元格操作 ![单元格操作](../_static/images/notebooks/cell-actions.png) 包含三个菜单：“运行”、“仪表板”和“编辑”：

![“运行”图标](../_static/images/notebooks/run-cell.png) — ![仪表板](../_static/images/dashboards/dashboard-demo-3.png) — ![编辑](../_static/images/notebooks/cmd-edit.png)

以及两个操作：隐藏 ![将单元格最小化](../_static/images/notebooks/cell-minimize.png) 和“删除” ![“删除”图标](../_static/images/clusters/delete-icon.png).

### <a name="add-a-cell"></a>添加单元格

若要添加单元格，请将鼠标指针悬停在顶部或底部的某个单元格上，然后单击 ![添加单元格](../_static/images/notebooks/add-cell.png) 图标，或访问最右侧的笔记本单元格菜单，单击 ![向下的脱字号](../_static/images/down-caret.png)，然后选择“添加上面的单元格”或“添加下面的单元格”。

### <a name="delete-a-cell"></a>删除单元格

转到最右侧的“单元格操作”菜单 ![单元格操作](../_static/images/notebooks/cell-actions.png) 并单击 ![“删除”图标](../_static/images/clusters/delete-icon.png) （删除）。

删除单元格时，默认情况下会显示一个用于确认删除操作的对话框。 若要禁用未来的确认对话框，请选中“不再显示此对话框”复选框，然后单击“确认”。 你还可以切换确认对话框设置，只需使用 ![“帐户”图标](../_static/images/account-settings/account-icon.png) >“用户设置”>“笔记本设置”中的“启用命令删除确认”选项即可。 

若要还原已删除的单元格，请选择“编辑 > 撤消删除单元格”或使用 (`Z`) 键盘快捷方式。

### <a name="cut-a-cell"></a>剪切单元格

转到最右侧的“单元格操作”菜单 ![单元格操作](../_static/images/notebooks/cell-actions.png)，单击 ![向下的脱字号](../_static/images/down-caret.png)，然后选择“剪切单元格”。

还可以使用 (`X`) 键盘快捷方式。

若要还原已删除的单元格，请选择“编辑 > 撤消剪切单元格”或使用 (`Z`) 键盘快捷方式。

### <a name="select-multiple-cells-or-all-cells"></a>选择多个单元格或所有单元格

可以分别对上一个和下一个单元格使用 **Shift** + **向上箭头** 或 **向下箭头** 来选择相邻的笔记本单元格。 可以复制、剪切、删除和粘贴多选的单元格。

若要选择所有单元格，请选择“编辑”>“选择所有单元格”或使用命令模式快捷方式 **Cmd+A** 。

### <a name="default-language"></a>默认语言

每个单元格的默认语言显示在笔记本名称旁边的 (<language>) 链接中。 在以下笔记本中，默认语言为 SQL。

> [!div class="mx-imgBorder"]
> ![笔记本默认语言](../_static/images/notebooks/toolbar.png)

若要更改默认语言，请执行以下操作：

1. 单击 (<language>) 链接。 此时会显示“更改默认语言”对话框。

   > [!div class="mx-imgBorder"]
   > ![笔记本默认语言](../_static/images/notebooks/change-default-language.png)

2. 从“默认语言”下拉列表中选择新语言。
3. 单击“更改”。
4. 为确保现有命令可继续正常工作，以前的默认语言的命令会自动带有语言 magic 命令前缀。

### <a name="mix-languages"></a><a id="language-magic"> </a><a id="mix-languages"> </a>混合语言

可以通过在单元格开头指定语言 magic 命令 `%<language>` 来重写默认语言。 支持的 magic 命令为：`%python`、`%r`、`%scala` 和 `%sql`。

> [!NOTE]
>
> 调用语言 magic 命令时，该命令会被调度到笔记本的[执行上下文](notebooks-manage.md#execution-context)中的 REPL。 用一种语言定义（并且因此位于该语言的 REPL 中）的变量在其他语言的 REPL 中不可用。 REPL 只能通过外部资源（例如 DBFS 中的文件或对象存储中的对象）共享状态。

笔记本还支持几个辅助 magic 命令：

* `%sh`：允许你在笔记本中运行 shell 代码。 若要在 shell 命令的退出状态为非零值的情况下使单元格发生失败，请添加 `-e` 选项。 此命令仅在 Apache Spark 驱动程序上运行，不在工作器上运行。 若要在所有节点上运行 shell 命令，请使用[初始化脚本](../clusters/init-scripts.md)。
* `%fs`：允许你使用 `dbutils` 文件系统命令。 请参阅 [dbutils](../data/databricks-file-system.md#dbfs-dbutils)。
* `%md`：允许你包括各种类型的文档，例如文本、图像以及数学公式和等式。 请参阅下一部分。

### <a name="include-documentation"></a>包括文档

若要在笔记本中包括文档，可以使用 `%md` magic 命令来标识 Markdown 标记。 包括的 Markdown 标记将呈现为 HTML。 例如，此 Markdown 代码片段包含一级标题的标记：

```md
%md # Hello This is a Title
```

它会呈现为 HTML 标题：

> [!div class="mx-imgBorder"]
> ![笔记本 HTML 标题](../_static/images/notebooks/title.png)

#### <a name="collapsible-headings"></a>可折叠的标题

在包含 Markdown 标题的单元格后显示的单元格可以折叠到标题单元格中。 下图显示了名为“标题 1”的一级标题，后面的两个单元格已折叠到其中。

> [!div class="mx-imgBorder"]
> ![折叠的单元格](../_static/images/notebooks/headings.png)

若要展开和折叠标题，请单击 **+** 和 **-** 。

另请参阅[隐藏和显示单元格内容](#hide-show-cell)。

#### <a name="link-to-other-notebooks"></a>链接到其他笔记本

可以使用相对路径链接到 Markdown 单元格中的其他笔记本或文件夹。 将定位点标记的 `href` 属性指定为相对路径（以 `$` 开头），然后遵循与 Unix 文件系统中的模式相同的模式：

```md
%md
<a href="$./myNotebook">Link to notebook in same folder as current notebook</a>
<a href="$../myFolder">Link to folder in parent folder of current notebook</a>
<a href="$./myFolder2/myNotebook2">Link to nested notebook</a>
```

#### <a name="display-images"></a>显示图像

若要显示在 [FileStore](../data/filestore.md#static-images) 中存储的图像，请使用以下语法：

```md
%md
![test](files/image.png)
```

例如，假设你在 FileStore 中有 Databricks 徽标图像文件：

```bash
dbfs ls dbfs:/FileStore/
```

```console
databricks-logo-mobile.png
```

在 Markdown 单元格中包括以下代码时：

> [!div class="mx-imgBorder"]
> ![Markdown 单元格中的图像](../_static/images/notebooks/image-code.png)

图像会呈现在单元格中：

> [!div class="mx-imgBorder"]
> ![呈现的图像](../_static/images/notebooks/image-render.png)

#### <a name="display-mathematical-equations"></a>显示数学等式

笔记本支持 [KaTeX](https://github.com/Khan/KaTeX/wiki)，用于显示数学公式和等式。 例如，

```md
%md
\\(c = \\pm\\sqrt{a^2 + b^2} \\)

\\(A{_i}{_j}=B{_i}{_j}\\)

$$c = \\pm\\sqrt{a^2 + b^2}$$

\\[A{_i}{_j}=B{_i}{_j}\\]
```

呈现为：

> [!div class="mx-imgBorder"]
> ![呈现的等式 1](../_static/images/notebooks/equations.png)

and

```md
%md
\\( f(\beta)= -Y_t^T X_t \beta + \sum log( 1+{e}^{X_t\bullet\beta}) + \frac{1}{2}\delta^t S_t^{-1}\delta\\)

where \\(\delta=(\beta - \mu_{t-1})\\)
```

呈现为：

> [!div class="mx-imgBorder"]
> ![呈现的等式 2](../_static/images/notebooks/equations2.png)

#### <a name="include-html"></a>包括 HTML

可以使用函数 `displayHTML` 在笔记本中包括 HTML。 请参阅[笔记本中的 HTML、D3 和 SVG](visualizations/html-d3-and-svg.md)，通过示例来了解如何执行此操作。

> [!NOTE]
>
> `displayHTML` iframe 是从域 `databricksusercontent.com` 提供的，iframe 沙盒包含 `allow-same-origin` 属性。 必须可在浏览器中访问 `databricksusercontent.com`。 如果它当前被企业网络阻止，IT 人员需要将它加入允许列表。

### <a name="command-comments"></a>命令注释

你可以使用命令注释与协作者进行讨论。

若要切换“注释”边栏，请单击笔记本右上方的“注释”按钮。

> [!div class="mx-imgBorder"]
> ![切换笔记本注释](../_static/images/notebooks/comments.png)

若要向命令添加注释，请执行以下操作：

1. 突出显示命令文本，然后单击注释气泡：

   > [!div class="mx-imgBorder"]
   > ![打开注释](../_static/images/notebooks/add-comment.png)

2. 添加你的注释，然后单击“注释”。

   > [!div class="mx-imgBorder"]
   > ![添加注释](../_static/images/notebooks/save-comment.png)

若要编辑、删除或回复某项注释，请单击该注释并选择一项操作。

> [!div class="mx-imgBorder"]
> ![编辑注释](../_static/images/notebooks/edit-comment.png)

### <a name="change-cell-display"></a><a id="cell-display"> </a><a id="change-cell-display"> </a>更改单元格显示

笔记本有三个显示选项：

* 标准视图：结果紧跟在代码单元格之后显示
* 仅显示结果：只显示结果
* 并排显示：代码和结果单元格并排显示，结果显示在右侧

转到“视图”菜单 ![视图菜单](../_static/images/notebooks/view-menu-icon.png) 以选择显示选项。

> [!div class="mx-imgBorder"]
> ![“并排显示”视图](../_static/images/notebooks/side-by-side.png)

### <a name="show-line-and-command-numbers"></a>显示行号和命令号

若要显示行号或命令号，请转到“视图”菜单 ![“视图”菜单](../_static/images/notebooks/view-menu-icon.png)，选择“显示行号”或“显示命令号”。 在它们显示后，你可以在同一菜单中再次隐藏它们。 你还可以使用键盘快捷方式 **Control+L** 来启用行号。

> [!div class="mx-imgBorder"]
> ![通过“视图”菜单显示行号或命令号](../_static/images/notebooks/notebook-view-menu.png)

> [!div class="mx-imgBorder"]
> ![在笔记本中启用的行号和命令号](../_static/images/notebooks/notebook-line-command-numbers.png)

如果启用行号或命令号，Databricks 会保存你的首选项，并在该浏览器的所有其他笔记本中显示它们。

单元格上方的命令号会链接到该特定命令。 如果单击某个单元格的命令号，则会更新 URL，使之定位到该命令。 如果要链接到笔记本中的特定命令，请右键单击命令号，然后选择“复制链接地址”。

### <a name="find-and-replace-text"></a>查找和替换文本

若要查找和替换笔记本中的文本，请选择“文件”>“查找和替换”。

> [!div class="mx-imgBorder"]
> ![查找和替换文本](../_static/images/notebooks/find-replace-in-dropdown.png)

当前的匹配项以橙色突出显示，所有其他的匹配项以黄色突出显示。

> [!div class="mx-imgBorder"]
> ![匹配文本](../_static/images/notebooks/find-replace-example.png)

可以通过单击“替换”来逐个替换匹配项。

可以在匹配项之间切换，方法是：单击“上一个”和“下一个”按钮，或按 Shift+Enter 和 Enter，分别转到上一个和下一个匹配项。   

通过单击 ![“删除”图标](../_static/images/clusters/delete-icon.png) 或按 **Esc** ，关闭查找和替换工具。

### <a name="autocomplete"></a>自动完成

可以使用 Azure Databricks 的自动完成功能，以便在单元格中输入代码段时自动完成这些代码段。 这可以减少那些需要记住的内容，最大限度地减少需要完成的键入量。 Azure Databricks 在笔记本中支持两种类型的自动完成：本地自动完成和服务器自动完成。

本地自动完成会完成笔记本中存在的单词。 服务器自动完成的功能更强大，因为它会针对定义的类型、类和对象以及 SQL 数据库和表名称访问群集。 若要激活服务器自动完成功能，必须附加[将笔记本附加到群集](notebooks-manage.md#attach)和[运行所有单元格](#notebook-run)功能，以便定义可完成的对象。

> [!IMPORTANT]
>
> 在执行命令的过程中，会阻止 R 笔记本中的服务器自动完成功能。

输入一个可完成的对象后按 **Tab** 键可触发自动完成。 例如，在定义和运行包含 `MyClass` 和 `instance` 的定义的单元后，`instance` 的方法就是可完成的方法。当你按 **Tab** 键时，会显示有效的完成操作的列表。

> [!div class="mx-imgBorder"]
> ![触发自动完成](../_static/images/notebooks/notebook-autocomplete-object.png)

类型完成采用的方式与 SQL 数据库和表名称的完成采用的方式相同。

![类型完成](../_static/images/notebooks/notebook-autocomplete-type.png) — — ![SQL 完成](../_static/images/notebooks/notebook-autocomplete-sql.png)

### <a name="format-sql"></a><a id="format-sql"> </a><a id="sql-format"> </a>设置 SQL 格式

Azure Databricks 提供的工具可用于在笔记本单元格中快速且轻松地设置 SQL 代码的格式。 这些工具减少了使代码带有格式的工作量，有助于在笔记本中强制实施相同的编码标准。

可通过以下方式触发格式化程序：

* **单个单元格**
  * 键盘快捷方式：按 **Cmd+Shift+F** 。
  * 命令上下文菜单：在 SQL 单元格的命令上下文下拉菜单中选择“设置 SQL 格式”。 此项仅在 SQL 笔记本单元格和具有 `%sql` [语言 magic](#language-magic) 的单元格中可见。

    > [!div class="mx-imgBorder"]
    > ![从命令上下文菜单设置 SQL 格式](../_static/images/notebooks/notebook-formatsql-cmd-context.png)

* **多个单元格**

  选择[多个 SQL 单元格](#select-multiple-cells-or-all-cells)，然后选择“编辑”>“设置 SQL 单元格格式”。 如果选择多个语言的单元格，则仅会设置 SQL 单元格的格式。 这包括那些使用 `%sql` 的单元格。

  > [!div class="mx-imgBorder"]
  > ![在“编辑”菜单中设置 SQL 格式](../_static/images/notebooks/notebook-formatsql-edit-menu.png)

下面是前面示例中进行格式设置后的第一个单元格：

> [!div class="mx-imgBorder"]
> ![设置 SQL 格式之后](../_static/images/notebooks/notebook-formatsql-after.png)

## <a name="run-notebooks"></a><a id="notebook-run"> </a><a id="run-notebooks"> </a>运行笔记本

此部分介绍如何运行一个或多个笔记本单元格。

### <a name="in-this-section"></a>本节内容：

* [惠?](#requirements)
* [运行单元格](#run-a-cell)
* [运行上方或下方的所有单元格](#run-all-above-or-below)
* [运行所有单元格](#run-all-cells)
* [查看每个单元格的多个输出](#view-multiple-outputs-per-cell)
* [Python 和 Scala 错误突出显示](#python-and-scala-error-highlighting)
* [通知](#notifications)
* [Databricks 顾问](#databricks-advisor)
* [从一个笔记本中运行另一个笔记本](#run-a-notebook-from-another-notebook)

### <a name="requirements"></a>要求

必须将笔记本[附加](notebooks-manage.md#attach)到群集。 如果群集未运行，则会在运行一个或多个单元格时启动该群集。

### <a name="run-a-cell"></a>运行单元格

在最右侧的“单元格操作”菜单 ![单元格操作](../_static/images/notebooks/cell-actions.png) 中，单击 ![“运行”图标](../_static/images/notebooks/run-cell.png) 并选择“运行单元格”，或者按 **Shift+Enter** 。

> [!IMPORTANT]
>
> 笔记本单元格（内容和输出）的最大大小为 16MB。

例如，尝试运行这个引用预定义的 `spark` [变量](notebooks-manage.md#variables)的 Python 代码片段。

```python
spark
```

然后，运行一些真实的代码：

```python
1+1 # => 2
```

> [!NOTE]
>
> 笔记本有多个默认设置：
>
> * 运行某个单元格时，笔记本会自动[附加](notebooks-manage.md#attach)到正在运行的群集，而不会进行提示。
> * 按 **Shift+Enter** 时，如果该单元格不可见，笔记本会自动滚动到下一个单元格。
>
> 若要更改这些设置，请选择 ![“帐户”图标](../_static/images/account-settings/account-icon.png) >“用户设置”>“笔记本设置”并配置相应的复选框。

### <a name="run-all-above-or-below"></a>运行上方或下方的所有单元格

若要运行某个单元格之前或之后的所有单元格，请转到最右侧的“单元格操作”菜单 ![单元格操作](../_static/images/notebooks/cell-actions.png)，单击 ![“运行”菜单](../_static/images/notebooks/run-menu.png) 并选择“运行上方的所有单元格”或“运行下方的所有单元格” 。

“运行下方的所有单元格”包括你所在的单元格。 “运行上方的所有单元格”不包括你所在的单元格。

### <a name="run-all-cells"></a><a id="run-all"> </a><a id="run-all-cells"> </a>运行所有单元格

若要运行笔记本中的所有单元格，请在笔记本工具栏中选择“全部运行”。

> [!IMPORTANT]
>
> 如果在同一笔记本中执行[装载和卸载](../data/databricks-file-system.md#mount-storage)步骤，请不要执行“全部运行”。 这可能导致出现争用情况并可能损坏装入点。

### <a name="view-multiple-outputs-per-cell"></a>查看每个单元格的多个输出

Python 笔记本以及非 Python 笔记本中的 `%python` 单元格支持每个单元格多个输出。

> [!div class="mx-imgBorder"]
> ![一个单元格中有多个输出](../_static/images/notebooks/multiple-cell-outputs.gif)

此功能需要 Databricks Runtime 7.1 或更高版本，并且在 Databricks Runtime 7.1 中默认处于禁用状态。 可以通过设置 `spark.databricks.workspace.multipleResults.enabled true` 来启用它。

### <a name="python-and-scala-error-highlighting"></a>Python 和 Scala 错误突出显示

Python 和 Scala 笔记本支持错误突出显示。 也就是说，系统会在单元格中突出显示引发错误的代码行。 此外，如果错误输出为堆栈跟踪，则引发错误的单元格会在堆栈跟踪中显示为指向该单元格的链接。 单击该链接可跳转到有问题的代码。

> [!div class="mx-imgBorder"]
> ![Python 错误突出显示](../_static/images/notebooks/notebook-python-error-highlighting.png)

> [!div class="mx-imgBorder"]
> ![Scala 错误突出显示](../_static/images/notebooks/notebook-scala-error-highlighting.png)

### <a name="notifications"></a>通知

通知会提醒你某些事件，例如，哪个命令当前正在[运行所有单元格](#run-all)阶段运行，哪些命令处于错误状态。 当笔记本显示多个错误通知时，第一个错误通知会有一个用于清除所有通知的链接。

> [!div class="mx-imgBorder"]
> ![笔记本通知](../_static/images/notebooks/notification.png)

默认情况下，笔记本通知处于启用状态。 可以在 ![“帐户”图标](../_static/images/account-settings/account-icon.png) >“用户设置”>“笔记本设置”下禁用它们。

### <a name="databricks-advisor"></a>Databricks 顾问

Databricks 顾问会在每次运行命令时自动分析命令，并在笔记本中显示相应的建议。 建议通知所提供的信息可帮助你提高工作负载的性能、降低成本以及避免常见错误。

#### <a name="view-advice"></a>查看建议

带有灯泡图标的蓝色框表示命令已有建议。 该框会显示不同建议的数目。

> [!div class="mx-imgBorder"]
> ![Databricks 建议](../_static/images/notebooks/advice-collapsed.png)

单击灯泡可展开框并查看建议。 一个或多个建议将变得可见。

> [!div class="mx-imgBorder"]
> ![查看建议](../_static/images/notebooks/advice-expanded.png)

单击“了解更多”链接可查看相关文档，了解与该建议相关的详细信息。

单击“不再显示此对话框”链接可隐藏该建议。 此类型的建议将不再显示。 此操作可[在“笔记本设置”中逆转](#advice-settings)。

再次单击灯泡可折叠建议框。

#### <a name="advice-settings"></a>建议设置

可以通过选择 ![“帐户”图标](../_static/images/account-settings/account-icon.png) >“用户设置”>“笔记本设置”或单击展开的建议框中的齿轮图标来访问“笔记本设置”页。

> [!div class="mx-imgBorder"]
> ![笔记本设置](../_static/images/notebooks/advice-notebook-settings.png)

切换“启用 Databricks 顾问”选项可启用或禁用通知。

如果当前隐藏了一个或多个类型的建议，则会显示“重置隐藏的建议”链接。 单击链接可使该建议类型再次可见。

### <a name="run-a-notebook-from-another-notebook"></a><a id="run"> </a><a id="run-a-notebook-from-another-notebook"> </a>从一个笔记本中运行另一个笔记本

可以通过使用 `%run <notebook>` magic 命令从一个笔记本中运行另一个笔记本。 这大致相当于本地计算机上的 Scala REPL 中的 `:load` 命令或 Python 中的 `import` 语句。 `<notebook>` 中定义的所有变量都会变得可在当前笔记本中使用。

`%run` 必须独自位于某个单元格中，因为它会以内联方式运行整个笔记本。

> [!NOTE]
>
> 不能使用 `%run` 来运行 Python 文件并将该文件中定义的实体 `import` 到笔记本中。 若要从 Python 文件导入，必须将文件打包到 Python 库，从该 Python 库创建 Azure Databricks [库](../libraries/index.md)，然后[将库安装到用于运行笔记本的群集](../libraries/cluster-libraries.md#install-libraries)。

#### <a name="example"></a>示例

假设你有 `notebookA` 和 `notebookB`。 `notebookA` 包含一个具有以下 Python 代码的单元格：

```python
x = 5
```

即使未在 `notebookB` 中定义 `x`，也可以在运行 `%run notebookA` 后访问 `notebookB` 中的 `x`。

```python
%run /Users/path/to/notebookA

print(x) # => 5
```

若要指定相对路径，请在其前面加上 `./` 或 `../`。 例如，如果 `notebookA` 和 `notebookB` 在同一目录中，则也可从相对路径运行它们。

```python
%run ./notebookA

print(x) # => 5
```

```python
%run ../someDirectory/notebookA # up a directory and into another

print(x) # => 5
```

有关笔记本之间的更复杂的交互，请参阅[笔记本工作流](notebook-workflows.md)。

## <a name="manage-notebook-state-and-results"></a><a id="manage-notebook-state-and-results"> </a><a id="notebook-state-results"> </a>管理笔记本状态和结果

[将笔记本附加到群集](notebooks-manage.md#attach)并[运行一个或多个单元格](#notebook-run)后，笔记本就会有状态并会显示结果。 此部分介绍如何管理笔记本状态和结果。

### <a name="in-this-section"></a>本节内容：

* [清除笔记本状态和结果](#clear-notebooks-state-and-results)
* [下载结果](#download-results)
* [下载单元格结果](#download-a-cell-result)
* [隐藏和显示单元格内容](#hide-and-show-cell-content)
* [笔记本隔离](#notebook-isolation)

### <a name="clear-notebooks-state-and-results"></a><a id="clear"> </a><a id="clear-notebooks-state-and-results"> </a>清除笔记本状态和结果

若要清除笔记本状态和结果，请单击笔记本工具栏中的“清除”，然后选择操作：

> [!div class="mx-imgBorder"]
> ![清除状态和结果](../_static/images/notebooks/clear-notebook.png)

### <a name="download-results"></a>下载结果

默认情况下已启用“下载结果”。 若要切换此设置，请参阅[管理从笔记本下载结果的功能](../administration-guide/workspace/notebooks.md#manage-download-results)。 如果“下载结果”处于禁用状态，则不会显示 ![下载结果](../_static/images/notebooks/download-results.png) 按钮。

### <a name="download-a-cell-result"></a>下载单元格结果

可以将包含表格输出的单元格结果下载到本地计算机。 单击单元格底部的 ![下载结果](../_static/images/notebooks/download-results.png) 按钮。

> [!div class="mx-imgBorder"]
> ![下载单元格结果](../_static/images/notebooks/download-result.png)

名为 `export.csv` 的 CSV 文件将下载到默认的下载目录。

#### <a name="download-full-results"></a>下载完整结果

默认情况下，Azure Databricks 返回一个数据帧的 1000 行。 超过 1000 行时，系统会将一个向下箭头 ![下拉按钮](../_static/images/button-down.png) 添加到 ![下载结果](../_static/images/notebooks/download-results.png) 按钮。  若要下载某个查询的所有结果，请执行以下操作：

1. 单击 ![下载结果](../_static/images/notebooks/download-results.png) 旁边的向下箭头，然后选择“下载完整结果”。

   > [!div class="mx-imgBorder"]
   > ![下载完整结果](../_static/images/notebooks/dbfs-download-full-results-1.png)

2. 选择“重新执行并下载”。

   > [!div class="mx-imgBorder"]
   > ![重新运行并下载结果](../_static/images/notebooks/dbfs-download-full-results-2.png)

   下载完整结果后，名为 `export.csv` 的 CSV 文件就会下载到本地计算机，此时会在 `/databricks-results` 文件夹中出现一个生成的文件夹，其中包含完整的查询结果。

   > [!div class="mx-imgBorder"]
   > ![下载的结果](../_static/images/notebooks/dbfs-databricks-downloaded-results.png)

### <a name="hide-and-show-cell-content"></a><a id="hide-and-show-cell-content"> </a><a id="hide-show-cell"> </a>隐藏和显示单元格内容

单元格内容包含单元格代码和运行单元格后的结果。 可以使用单元格右上角的“单元格操作”菜单 ![单元格操作](../_static/images/notebooks/cell-actions.png) 隐藏和显示单元格代码和结果。

若要隐藏单元格代码，请执行以下操作：

* 单击 ![向下的脱字号](../_static/images/down-caret.png) 并选择“隐藏代码”

若要隐藏和显示单元格结果，请执行下列任一操作：

* 单击 ![向下的脱字号](../_static/images/down-caret.png) 并选择“隐藏结果”
* Select ![将单元格最小化](../_static/images/notebooks/cell-minimize.png)
* 键入 **Esc > Shift + o**

若要显示隐藏的单元格代码或结果，请单击“显示”链接：

> [!div class="mx-imgBorder"]
> ![显示隐藏的代码和结果](../_static/images/notebooks/notebook-cell-show.png)

另请参阅[可折叠的标题](#collapsible-headings)。

### <a name="notebook-isolation"></a>笔记本隔离

笔记本隔离是指变量和类在笔记本之间的可见性。 Azure Databricks 支持两种类型的隔离：

* 变量和类隔离
* Spark 会话隔离

> [!NOTE]
>
> 由于附加到同一群集的所有笔记本都在同一群集 VM 上执行，因此即使启用了 Spark 会话隔离，也不能保证群集内的用户隔离。

#### <a name="variable-and-class-isolation"></a>变量和类隔离

变量和类仅在当前笔记本中可用。 例如，附加到同一群集的两个笔记本可以定义具有同一名称的变量和类，但这些对象是不同的。

若要定义一个对附加到同一群集的所有笔记本均可见的类，请在[包单元格](package-cells.md)中定义该类。 然后，可以使用完全限定的名称来访问该类，这与访问附加的 Scala 或 Java 库中的类是相同的。

#### <a name="spark-session-isolation"></a><a id="session_isolation"> </a><a id="spark-session-isolation"> </a>Spark 会话隔离

附加到运行 Apache Spark 2.0.0 及更高版本的群集的每个笔记本都有一个称为 `spark` 的[预定义变量](notebooks-manage.md#variables)，该变量表示 `SparkSession`。 `SparkSession` 是使用 Spark API 以及设置运行时配置的入口点。

默认情况下已启用 Spark 会话隔离。 你还可以使用全局临时视图跨笔记本共享临时视图。 请参阅[创建视图](../spark/latest/spark-sql/language-manual/create-view.md#create-view)。 若要禁用 Spark 会话隔离，请在 [Spark 配置](../clusters/configure.md#spark-config)中将 `spark.databricks.session.share` 设置为 `true`。

> [!IMPORTANT]
>
> 将 `spark.databricks.session.share` 设置为 true 会中断流式处理笔记本单元格和流式处理作业所使用的监视。 具体而言：
>
> * 不会显示流式处理单元格中的[图](../getting-started/spark/streaming.md#start-stream)。
> * 只要流在运行，就不会阻止作业（它们在“成功地”完成后就会停止流）。
> * 系统不会监视作业中的流是否已终止， 而必须由你来手动调用 `awaitTermination()`。
> * 对流式处理数据帧调用[显示函数](visualizations/index.md#display-function)将不起作用。

使用其他语言来触发命令的单元格（即，使用 `%scala`、`%python`、`%r` 和 `%sql` 的单元格）以及包含其他笔记本的单元格（即使用 `%run` 的单元格）都是当前笔记本的一部分。 因此，这些单元格与其他笔记本单元格位于相同的会话中。 相比之下，[笔记本工作流](notebook-workflows.md)使用独立的 `SparkSession` 来运行笔记本，这意味着在此类笔记本中定义的临时视图在其他笔记本中不可见。

## <a name="version-control"></a>版本控制

Azure Databricks 为笔记本提供了基本版本控制功能。 你可以执行以下有关修订的操作：添加注释、还原和删除修订，以及清除修订历史记录。

若要访问笔记本修订版本，请单击笔记本工具栏右上方的“修订历史记录”。

### <a name="in-this-section"></a>本节内容：

* [添加注释](#add-a-comment)
* [还原修订版本](#restore-a-revision)
* [删除修订版本](#delete-a-revision)
* [清除修订历史记录](#clear-a-revision-history)
* [Git 版本控制](#git-version-control)

### <a name="add-a-comment"></a>添加注释

若要将注释添加到最新修订版本，请执行以下操作：

1. 单击修订版本。
2. 单击“立即保存”链接。

   > [!div class="mx-imgBorder"]
   > ![保存注释](../_static/images/notebooks/revision-comment.png)

3. 在“保存笔记本修订版本”对话框中，输入注释。
4. 单击“保存”  。 笔记本修订版本将连同输入的注释一起保存。

### <a name="restore-a-revision"></a>还原修订版本

若要还原某个修订版本，请执行以下操作：

1. 单击修订版本。
2. 单击“还原此修订版本”。

   > [!div class="mx-imgBorder"]
   > ![还原修订版本](../_static/images/notebooks/restore-revision.png)

3. 单击“确认”  。 所选修订版本将成为笔记本的最新版本。

### <a name="delete-a-revision"></a>删除修订版本

若要删除笔记本的修订条目，请执行以下操作：

1. 单击修订版本。
2. 单击回收站图标 ![回收站](../_static/images/workspace/trash-icon.png).

   > [!div class="mx-imgBorder"]
   > ![删除修订版本](../_static/images/notebooks/delete-revision.png)

3. 单击“是，擦除”。 将从笔记本的修订历史记录中删除所选的修订版本。

### <a name="clear-a-revision-history"></a>清除修订历史记录

若要清除笔记本的修订历史记录，请执行以下操作：

1. 选择“文件”>“清除修订历史记录”。
2. 单击“是，清除”。 此时会清除笔记本修订历史记录。

   > [!WARNING]
   >
   > 清除后，修订历史记录将无法恢复。

### <a name="git-version-control"></a>Git 版本控制

Azure Databricks 还集成了以下基于 Git 的版本控制工具：

* [GitHub 版本控制](github-version-control.md)
* [Bitbucket 云版本控制](bitbucket-cloud-version-control.md)
* [Azure DevOps Services 版本控制](azure-devops-services-version-control.md)