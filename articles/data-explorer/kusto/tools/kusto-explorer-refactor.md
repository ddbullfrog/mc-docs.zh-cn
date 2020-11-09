---
title: Kusto 资源管理器代码重构 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 Kusto 资源管理器代码重构。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 06/05/2019
ms.date: 09/30/2020
ms.openlocfilehash: f96755698b83e43efbb20a6f1ad3e7d2db0c1b9c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103662"
---
# <a name="kusto-explorer-code-refactoring"></a>Kusto 资源管理器代码重构

与其他 IDE 类似，Kusto.Explorer 为 KQL 查询编辑和重构提供了几个功能。

## <a name="rename-variable-or-column-name"></a>变量名或列名重命名

单击“查询编辑器”窗口中的 `Ctrl`+`R`、`Ctrl`+`R`，可重命名当前选定的符号。

请参阅下面的快照，其中展示了相应体验：

![动画 GIF 显示了“查询编辑器”窗口中正在重命名的变量。三个事件同时更换为新名称。](./Images/kusto-explorer-refactor/ke-refactor-rename.gif "refactor-rename")

## <a name="extract-scalars-as-let-expressions"></a>将标量作为 `let` 表达式提取

通过单击 `Alt`+`Ctrl`+`M`，可以将当前选定的文本提升为 `let` 表达式。 

![动画 GIF。查询编辑器指针从文本表达式开始。然后显示一条 let 语句，该语句将该文本值设置为新变量。](./Images/kusto-explorer-refactor/ke-extract-as-let-literal.gif "extract-as-let-literal")

## <a name="extract-tabular-statements-as-let-expressions"></a>将表格语句提取为 `let` 表达式

还可以通过选择表格表达式的文本，然后单击 `Alt`+`Ctrl`+`M`，将其提升为 `let` 语句。 

![动画 GIF。在查询编辑器中已选中表格表达式。然后显示一个 let 语句，该语句将该表格表达式设置为新变量。](./Images/kusto-explorer-refactor/ke-extract-as-let-tabular.gif "extract-as-let-tabular")
