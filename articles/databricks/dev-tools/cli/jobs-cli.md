---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/30/2020
title: 作业 CLI - Azure Databricks
description: 了解如何使用 Databricks 作业命令行界面。
ms.openlocfilehash: 71091a848c87681da9890adf7750302e6350ba21
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937641"
---
# <a name="jobs-cli"></a>作业 CLI

通过将 Databricks 作业 CLI 子命令追加到 `databricks jobs` 后面来运行作业 CLI 子命令，而通过将作业运行命令追加到 `databricks runs` 后面来运行作业运行命令。

```bash
databricks jobs -h
```

```
Usage: databricks jobs [OPTIONS] COMMAND [ARGS]...

  Utility to interact with jobs.

  Job runs are handled by ``databricks runs``.

Options:
  -v, --version  [VERSION]
  -h, --help     Show this message and exit.

Commands:
  create   Creates a job.
    Options:
      --json-file PATH            File containing JSON request to POST to /api/2.0/jobs/create.
      --json JSON                 JSON string to POST to /api/2.0/jobs/create.
  delete   Deletes a job.
    Options:
      --job-id JOB_ID             Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#job/$JOB_ID. [required]
  get      Describes the metadata for a job.
    Options:
    --job-id JOB_ID               Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#job/$JOB_ID. [required]
  list     Lists the jobs in the Databricks Job Service.
  reset    Resets (edits) the definition of a job.
    Options:
      --job-id JOB_ID             Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#job/$JOB_ID. [required]
      --json-file PATH            File containing JSON request to POST to /api/2.0/jobs/create.
      --json JSON                 JSON string to POST to /api/2.0/jobs/create.
  run-now  Runs a job with optional per-run parameters.
    Options:
      --job-id JOB_ID             Can be found in the URL at https://<databricks-instance>/#job/$JOB_ID. [required]
      --jar-params JSON           JSON string specifying an array of parameters. i.e. '["param1", "param2"]'
      --notebook-params JSON      JSON string specifying a map of key-value pairs. i.e. '{"name": "john doe", "age": 35}'
      --python-params JSON        JSON string specifying an array of parameters. i.e. '["param1", "param2"]'
      --spark-submit-params JSON  JSON string specifying an array of parameters. i.e. '["--class", "org.apache.spark.examples.SparkPi"]'
```

```bash
databricks runs -h
```

```
Usage: databricks runs [OPTIONS] COMMAND [ARGS]...

  Utility to interact with job runs.

Options:
  -v, --version  [VERSION]
  -h, --help     Show this message and exit.

Commands:
  cancel  Cancels a run.
    Options:
        --run-id RUN_ID  [required]
  get     Gets the metadata about a run in JSON form.
    Options:
      --run-id RUN_ID  [required]

  get-output Gets the output of a run. See [_](/api/latest/jobs.html#runs-get-output).
  list    Lists job runs.
  submit  Submits a one-time run.
    Options:
      --json-file PATH  File containing JSON request to POST to /api/2.0/jobs/runs/submit.
      --json JSON       JSON string to POST to /api/2.0/jobs/runs/submit.
```

## <a name="list-and-find-jobs"></a>列出并查找作业

`databricks jobs list` 命令有两种输出格式：`JSON` 和 `TABLE`。
默认情况下将输出 `TABLE` 格式，并返回一个两列的表（作业 ID、作业名称）。

若要按名称查找作业，请运行：

```bash
databricks jobs list | grep "JOB_NAME"
```

## <a name="copy-a-job"></a>复制作业

> [!NOTE]
>
> 此示例需要程序 [jq](index.md#jq)。

```bash
SETTINGS_JSON=$(databricks jobs get --job-id 284907 | jq .settings)
# JQ Explanation:
#   - peek into top level `settings` field.
databricks jobs create --json "$SETTINGS_JSON"
```

## <a name="delete-untitled-jobs"></a>删除“无标题”作业

```bash
databricks jobs list --output json | jq '.jobs[] | select(.settings.name == "Untitled") | .job_id' | xargs -n 1 databricks jobs delete --job-id
# Explanation:
#   - List jobs in JSON.
#   - Peek into top level `jobs` field.
#   - Select only jobs with name equal to "Untitled"
#   - Print those job IDs out.
#   - Invoke `databricks jobs delete --job-id` once per row with the $job_id appended as an argument to the end of the command.
```