---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/21/2020
title: 群集 CLI - Azure Databricks
description: 了解如何使用 Databricks 群集命令行界面。
ms.openlocfilehash: 645e65eb6fa64d765efa022dde870ed19521ebae
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937633"
---
# <a name="clusters-cli"></a>群集 CLI

可以通过将 Databricks 群集 CLI 子命令追加到 `databricks clusters` 后面来运行这些命令。

```bash
databricks clusters -h
```

```
Usage: databricks clusters [OPTIONS] COMMAND [ARGS]...

  Utility to interact with Databricks clusters.

Options:
  -v, --version  [VERSION]
  -h, --help     Show this message and exit.

Commands:
  create           Creates a Databricks cluster.
    Options:
      --json-file PATH  File containing JSON request to POST to /api/2.0/clusters/create.
      --json JSON       JSON string to POST to /api/2.0/clusters/create.
  delete           Removes a Databricks cluster.
    Options:
      --cluster-id CLUSTER_ID Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#/setting/clusters/$CLUSTER_ID/configuration.
  edit             Edits a Databricks cluster.
    Options:
      --json-file PATH  File containing JSON request to POST to /api/2.0/clusters/edit.
      --json JSON       JSON string to POST to /api/2.0/clusters/edit.
  events Gets events for a Spark cluster.
    Options:
      --cluster-id CLUSTER_ID  Can be found in the URL at https://<databricks-instance>/#/setting/clusters/$CLUSTER_ID/configuration.  [required]
      --start-time TEXT        The start time in epoch milliseconds. If
                               unprovided, returns events starting from the
                               beginning of time.
      --end-time TEXT          The end time in epoch milliseconds. If unprovided,
                               returns events up to the current time
      --order TEXT             The order to list events in; either ASC or DESC.
                               Defaults to DESC (most recent first).
      --event-type TEXT        An event types to filter on (specify multiple event
                               types by passing the --event-type option multiple
                               times). If empty, all event types are returned.
      --offset TEXT            The offset in the result set. Defaults to 0 (no
                               offset). When an offset is specified and the
                               results are requested in descending order, the
                               end_time field is required.
      --limit TEXT             The maximum number of events to include in a page
                               of events. Defaults to 50, and maximum allowed
                               value is 500.
      --output FORMAT          can be "JSON" or "TABLE". Set to TABLE by default.
  get              Retrieves metadata about a cluster.
    Options:
      --cluster-id CLUSTER_ID Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#/setting/clusters/$CLUSTER_ID/configuration.
  list             Lists active and recently terminated clusters.
    Options:
      --output FORMAT          JSON or TABLE. Set to TABLE by default.
  list-node-types  Lists node types for a cluster.
  list-zones       Lists zones where clusters can be created.
  permanent-delete Permanently deletes a cluster.
    Options:
      --cluster-id CLUSTER_ID  Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#/setting/clusters/$CLUSTER_ID/configuration.
  resize           Resizes a Databricks cluster given its ID.
    Options:
      --cluster-id CLUSTER_ID  Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#/setting/clusters/$CLUSTER_ID/configuration.
      --num-workers INTEGER    Number of workers. [required]
  restart          Restarts a Databricks cluster.
    Options:
      --cluster-id CLUSTER_ID  Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#/setting/clusters/$CLUSTER_ID/configuration.
  spark-versions   Lists possible Databricks Runtime versions.
  start            Starts a terminated Databricks cluster.
    Options:
      --cluster-id CLUSTER_ID  Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#/setting/clusters/$CLUSTER_ID/configuration.
```

## <a name="list-runtime-versions"></a>列出运行时版本

```bash
databricks clusters spark-versions
```

## <a name="list-node-types"></a>列出节点类型

```bash
databricks clusters list-node-types
```