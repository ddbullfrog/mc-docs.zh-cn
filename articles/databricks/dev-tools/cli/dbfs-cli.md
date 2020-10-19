---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/30/2020
title: DBFS CLI - Azure Databricks
description: 了解如何使用 Databricks DBFS 命令行界面。
ms.openlocfilehash: 9e9f90fd9032cb72b7f78b810265070308cbb94d
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937636"
---
# <a name="dbfs-cli"></a>DBFS CLI

运行 Databricks DBFS CLI 命令，将其追加到 `databricks fs`（或别名 `dbfs`），并在所有 DBFS 路径前加上 `dbfs:/`。

```bash
databricks fs -h
```

```
Usage: databricks fs [OPTIONS] COMMAND [ARGS]...

  Utility to interact with DBFS. DBFS paths are all prefixed
  with dbfs:/. Local paths can be absolute or local.

Options:
  -v, --version
  -h, --help     Show this message and exit.

Commands:
  cat        Shows the contents of a file. Does not work for directories.
  configure
  cp         Copies files to and from DBFS.
    Options:
      -r, --recursive
      --overwrite     Overwrites files that exist already.
  ls         Lists files in DBFS.
    Options:
      --absolute      Displays absolute paths.
      -l              Displays full information including size and file type.
  mkdirs     Makes directories in DBFS.
  mv         Moves a file between two DBFS paths.
  rm         Removes files from DBFS.
    Options:
      -r, --recursive
```

对于列出、移动或删除超过 1 万个文件的操作，强烈建议不要使用 DBFS CLI。

* `list` 操作 (`databricks fs ls`) 会在大约 60 秒后超时。
* `move` 操作 (`databricks fs mv`) 会在大约 60 秒后超时，可能导致只有一部分数据被移动。
* `delete` 操作 (`databricks fs rm`) 会以增量方式删除成批的文件。

建议你使用[文件系统实用工具](../databricks-utils.md#dbutils-fs)在群集的上下文中执行此类操作。 `dbutils.fs` 涵盖 DBFS REST API 的功能范围，但仅限笔记本内部。 使用笔记本运行此类操作可提供更好的控制（例如选择性删除）和可管理性，并可自动执行定期作业。

## <a name="copy-a-file-to-dbfs"></a>将文件复制到 DBFS

```bash
dbfs cp test.txt dbfs:/test.txt
# Or recursively
dbfs cp -r test-dir dbfs:/test-dir
```

## <a name="copy-a-file-from-dbfs"></a>从 DBFS 复制文件

```bash
dbfs cp dbfs:/test.txt ./test.txt
# Or recursively
dbfs cp -r dbfs:/test-dir ./test-dir
```