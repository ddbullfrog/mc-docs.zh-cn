---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 实例池 CLI - Azure Databricks
description: 了解如何使用 Databricks 池命令行界面。
ms.openlocfilehash: 5ca2ff8c3f97e1f01963af47be1787f18783d828
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937831"
---
# <a name="instance-pools-cli"></a>实例池 CLI

> [!NOTE]
>
> 池 CLI 需要 Databricks CLI 0.9.0 或更高版本。

可以通过将子命令追加到 `databricks instance-pools` 后面来运行子命令。

```bash
databricks instance-pools -h
```

```
Usage: databricks instance-pools [OPTIONS] COMMAND [ARGS]...

  Utility to interact with Databricks instance pools.

Options:
  -v, --version  [VERSION]
  -h, --help     Show this message and exit.

Commands:
  create           Creates a Databricks instance pool.
    Options:
      --json-file PATH         File containing JSON request to POST to /api/2.0/cluster-pools/create.
      --json JSON              JSON string to POST to /api/2.0/cluster-pools/create.
  delete           Deletes a Databricks instance pool.
    Options:
  get              Retrieves metadata about an instance pool.
    Options:
      --instance-pool-id INSTANCE_POOL_ID Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#/setting/clusters/instance-pools/view/$INSTANCE_POOL_ID.
  list             Lists active instance pools with the stats of the pools.
    Options:
      --output FORMAT          JSON or TABLE. Set to TABLE by default.
  edit            Edits a Databricks instance pool
    Options:
      --json-file PATH         File containing JSON request to POST to /api/2.0/cluster-pools/create.
      --json JSON              JSON string to POST to /api/2.0/cluster-pools/create.
```