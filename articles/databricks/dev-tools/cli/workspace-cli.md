---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 工作区 CLI - Azure Databricks
description: 了解如何使用 Databricks 工作区命令行接口。
ms.openlocfilehash: f33692e5096ffc21bb2e40a2b194470d58515102
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937781"
---
# <a name="workspace-cli"></a>工作区 CLI

可以通过将 Databricks 工作区 CLI 子命令追加到 `databricks workspace` 之后来运行这些命令。

```bash
databricks workspace -h
```

```
Usage: databricks workspace [OPTIONS] COMMAND [ARGS]...

  Utility to interact with the Databricks workspace. Workspace paths must be
  absolute and be prefixed with `/`.

Common Options:
  -v, --version  [VERSION]
  -h, --help     Show this message and exit.

Commands:
  delete      Deletes objects from the Databricks workspace. rm and delete are synonyms.
    Options:
        -r, --recursive
  export      Exports a file from the Databricks workspace.
    Options:
      -f, --format FORMAT      SOURCE, HTML, JUPYTER, or DBC. Set to SOURCE by default.
      -o, --overwrite          Overwrites file with the same name as a Workspace file.
  export_dir  Recursively exports a directory from the Databricks workspace.
    Options:
      -o, --overwrite          Overwrites local files with the same names as Workspace files.
  import      Imports a file from local to the Databricks workspace.
    Options:
      -l, --language LANGUAGE  SCALA, PYTHON, SQL, R  [required]
      -f, --format FORMAT      SOURCE, HTML, JUPYTER, or DBC. Set to SOURCE by default.
      -o, --overwrite          Overwrites workspace files with the same names as local files.
  import_dir  Recursively imports a directory to the Databricks workspace.

    Only directories and files with the extensions .scala, .py, .sql, .r, .R,
    .ipynb are imported. When imported, these extensions are stripped off
    the name of the notebook.

    Options:
      -o, --overwrite          Overwrites workspace files with the same names as local files.
      -e, --exclude-hidden-files
  list        Lists objects in the Databricks workspace. ls and list are synonyms.
    Options:
      --absolute               Displays absolute paths.
      -l                       Displays full information including ObjectType, Path, Language
  ls          Lists objects in the Databricks workspace. ls and list are synonyms.
    Options:
      --absolute               Displays absolute paths.
      -l                       Displays full information including ObjectType, Path, Language
  mkdirs      Makes directories in the Databricks workspace.
  rm          Deletes objects from the Databricks workspace. rm and delete are synonyms.
    Options:
        -r, --recursive
```

## <a name="list-workspace-files"></a>列出工作区文件

```bash
databricks workspace ls /Users/example@databricks.com
```

```
Usage Logs ETL
Common Utilities
guava-21.0
```

## <a name="import-a-local-directory-of-notebooks"></a>导入笔记本的本地目录

`databricks workspace import_dir` 命令以递归方式将目录从本地文件系统导入到工作区。 仅导入扩展名为 `.scala`、`.py`、`.sql`、`.r`、`.R` 的目录和文件。
导入后，将从笔记本名称中删除这些扩展名。

若要覆盖目标路径上的现有笔记本，请添加标志 `-o`。

```bash
tree
```

```
.
├── a.py
├── b.scala
├── c.sql
├── d.R
└── e
```

```bash
databricks workspace import_dir . /Users/example@databricks.com/example
```

```
./a.py -> /Users/example@databricks.com/example/a
./b.scala -> /Users/example@databricks.com/example/b
./c.sql -> /Users/example@databricks.com/example/c
./d.R -> /Users/example@databricks.com/example/d
```

```bash
databricks workspace ls /Users/example@databricks.com/example -l
```

```
NOTEBOOK   a  PYTHON
NOTEBOOK   b  SCALA
NOTEBOOK   c  SQL
NOTEBOOK   d  R
DIRECTORY  e
```

## <a name="export-a-workspace-folder-to-the-local-filesystem"></a>将工作区文件夹导出到本地文件系统

可以将笔记本文件夹从工作区导出到本地文件系统。 若要执行此操作，请运行：

```bash
databricks workspace export_dir /Users/example@databricks.com/example .
```