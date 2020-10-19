---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/30/2020
title: 堆栈 CLI - Azure Databricks
description: 了解如何使用 Databricks 堆栈命令行界面。
ms.openlocfilehash: 9d70c7b1aa92093153a54c42f52debbb0499fc2c
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937780"
---
# <a name="stack-cli"></a>Stack CLI

> [!IMPORTANT]
>
> 此功能以 [Beta 版本](../../release-notes/release-types.md)提供。

> [!NOTE]
>
> 堆栈 CLI 需要 Databricks CLI 0.8.3 或更高版本。

堆栈 CLI 提供一种管理 Azure Databricks 资源（例如作业、笔记本和 DBFS 文件）堆栈的方法。 可以在本地存储笔记本和 DBFS 文件，并创建一个堆栈配置 JSON 模板，用于定义从本地文件到 Azure Databricks 工作区路径的映射，以及运行笔记本的作业的配置。

结合使用堆栈 CLI 和堆栈配置 JSON 模板来部署和管理堆栈。

可以通过将 Databricks 堆栈 CLI 子命令附加到 `databricks stack` 来运行它们。

```bash
databricks stack --help
```

```
Usage: databricks stack [OPTIONS] COMMAND [ARGS]...

  [Beta] Utility to deploy and download Databricks resource stacks.

Options:
  -v, --version   [VERSION]
  --debug         Debug Mode. Shows full stack trace on error.
  --profile TEXT  CLI connection profile to use. The default profile is
                  "DEFAULT".
  -h, --help      Show this message and exit.

Commands:
  deploy    Deploy a stack of resources given a JSON configuration of the stack
    Usage: databricks stack deploy [OPTIONS] CONFIG_PATH
    Options:
       -o, --overwrite  Include to overwrite existing workspace notebooks and DBFS
                        files  [default: False]
  download  Download workspace notebooks of a stack to the local filesystem
            given a JSON stack configuration template.
    Usage: databricks stack download [OPTIONS] CONFIG_PATH
    Options:
       -o, --overwrite  Include to overwrite existing workspace notebooks in the
                        local filesystem   [default: False]
```

## <a name="deploy-a-stack-to-a-workspace"></a>将堆栈部署到工作区

此子命令部署堆栈。 请参阅[堆栈设置](#stacksetup)，了解如何设置堆栈。

```bash
databricks stack deploy ./config.json
```

[堆栈配置 JSON 模板](#stackconfigtemplate)提供 `config.json` 的示例。

## <a name="download-stack-notebook-changes"></a>下载堆栈笔记本更改

此子命令下载堆栈的笔记本。

```bash
databricks stack download ./config.json
```

## <a name="examples"></a>示例

### <a name="stack-setup"></a><a id="stack-setup"> </a><a id="stacksetup"> </a>堆栈设置

## <a name="file-structure-of-an-example-stack"></a>示例堆栈的文件结构

```bash
tree
```

```
.
├── notebooks
|   ├── common
|   |   └── notebook.scala
|   └── config
|       ├── environment.scala
|       └── setup.sql
├── lib
|   └── library.jar
└── config.json
```

此示例堆栈在 `notebooks/common/notebook.scala` 中包含一个主笔记本，在 `notebooks/config` 文件夹中包含配置笔记本。 `lib/library.jar` 中有堆栈的 JAR 库依赖项。 `config.json` 是堆栈的堆栈配置 JSON 模板。 这就是传递给堆栈 CLI 的用于部署堆栈的内容。

## <a name="stack-configuration-json-template"></a><a id="stack-configuration-json-template"> </a><a id="stackconfigtemplate"> </a>堆栈配置 JSON 模板

堆栈配置模板描述堆栈配置。

```bash
cat config.json
```

```json
{
  "name": "example-stack",
  "resources": [
  {
    "id": "example-workspace-notebook",
    "service": "workspace",
    "properties": {
      "source_path": "notebooks/common/notebook.scala",
      "path": "/Users/example@example.com/dev/notebook",
      "object_type": "NOTEBOOK"
    }
  },
  {
    "id": "example-workspace-config-dir",
    "service": "workspace",
    "properties": {
      "source_path": "notebooks/config",
      "path": "/Users/example@example.com/dev/config",
      "object_type": "DIRECTORY"
    }
  },
  {
    "id": "example-dbfs-library",
    "service": "dbfs",
    "properties": {
      "source_path": "lib/library.jar",
      "path": "dbfs:/tmp/lib/library.jar",
      "is_dir": false
    }
  },
    {
      "id": "example-job",
      "service": "jobs",
      "properties": {
        "name": "Example Stack CLI Job",
        "new_cluster": {
          "spark_version": "5.0.x-scala2.11",
          "node_type_id": "Standard_DS3_v2",
          "num_workers": 3
        },
        "timeout_seconds": 7200,
        "max_retries": 1,
        "notebook_task": {
          "notebook_path": "/Users/example@example.com/dev/notebook"
        },
        "libraries": [
          {
            "jar": "dbfs:/tmp/lib/library.jar"
          }
        ]
      }
    }
  ]
}
```

每个作业、工作区笔记本、工作区目录、DBFS 文件或 DBFS 目录都定义为 [ResourceConfig](#resourceconfig)。 代表工作区或 DBFS 资产的每个 `ResourceConfig` 都包含一个从本地文件或目录 (`source_path`) 到其在工作区中或 DBFS 中的位置 (`path`) 的映射。

[堆栈配置模板架构](#stackconfiguration)概述了堆栈配置模板的架构。

### <a name="deploy-a-stack"></a><a id="deploy-a-stack"> </a><a id="stackdeployment"> </a>部署堆栈

使用 `databricks stack deploy <configuration-file>` 命令部署堆栈。

```bash
databricks stack deploy ./config.json
```

在堆栈部署过程中，会将 DBFS 和工作区资产上传到 Azure Databricks 工作区，并创建作业。

在堆栈部署时，用于部署的 [StackStatus](#stackstatus) JSON 文件与名称相同的堆栈配置模板保存在同一目录中，并在 `.json` 扩展名前添加 `deployed`：（例如 `./config.deployed.json`）。 堆栈 CLI 使用此文件来跟踪工作区上以前部署的资源。

[堆栈状态架构](#stackstatusschema)概述了堆栈配置的架构。

> [!IMPORTANT]
>
> 不要尝试编辑或移动堆栈状态文件。 如果收到有关堆栈状态文件的任何错误，请删除该文件，然后尝试重新部署。

```bash
./config.deployed.json
```

```json
{
  "cli_version": "0.8.3",
  "deployed_output": [
    {
      "id": "example-workspace-notebook",
      "databricks_id": {
        "path": "/Users/example@example.com/dev/notebook"
      },
      "service": "workspace"
    },
    {
      "id": "example-workspace-config-dir",
      "databricks_id": {
        "path": "/Users/example@example.com/dev/config"
      },
      "service": "workspace"
    },
    {
      "id": "example-dbfs-library",
      "databricks_id": {
        "path": "dbfs:/tmp/lib/library.jar"
      },
      "service": "dbfs"
    },
    {
      "id": "example-job",
      "databricks_id": {
        "job_id": 123456
      },
      "service": "jobs"
    }
  ],
  "name": "example-stack"
}
```

## <a name="data-structures"></a>数据结构

### <a name="in-this-section"></a>本节内容：

* [堆栈配置模板架构](#stack-configuration-template-schema)

### <a name="stack-configuration-template-schema"></a><a id="stack-configuration-template-schema"> </a><a id="stackconfiguration"> </a>堆栈配置模板架构

## <a name="stackconfig"></a>StackConfig

这些是堆栈配置模板的外围字段。 所有字段都是必填字段。

| 字段名称       | 类型                                      | 说明                                                                                                             |
|------------------|-------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| name             | `STRING`                                  | 堆栈的名称。                                                                                                  |
| resources        | [ResourceConfig](#resourceconfig) 列表 | Azure Databricks 中的资产。 资源与三个服务（REST API 命名空间）相关：工作区、作业和 dbfs。 |

## <a name="resourceconfig"></a>ResourceConfig

每个 `ResourceConfig` 的字段。 所有字段都是必填字段。

| 字段名称       | 类型                                      | 说明                                                                                    |
|------------------|-------------------------------------------|------------------------------------------------------------------------------------------------|
| id               | `STRING`                                  | 资源的唯一 ID。 强制执行 ResourceConfig 的唯一性。                        |
| 服务          | [ResourceService](#resourceservice)       | 资源运行时所在的 REST API 服务。 `jobs`、<br>`workspace` 或 `dbfs` 中的一项。 |
| properties       | [ResourceProperties](#resourceproperties) | 其中的字段根据 `ResourceConfig` 服务而异。                       |

## <a name="resourceproperties"></a>ResourceProperties

[ResourceService](#resourceservice) 提供的资源属性。 这些字段被分类为 Azure Databricks REST API 中已使用或未使用的字段。 列出的所有字段都是必需的。

| 服务              | 堆栈 CLI 中使用的 REST API 中的字段                                                                                                                                                                                                                                                                                                                     | 仅在堆栈 CLI 中使用的字段                                                                                                                                                |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 工作区            | path：`STRING`- 笔记本或目录的远程工作区路径。 （例如 `/Users/example@example.com/notebook`)<br><br>object_type：[ObjectType](../api/latest/workspace.md#workspaceobjecttype)- 笔记本对象类型。 只能是 `NOTEBOOK` 或 `DIRECTORY`。                                                                                                     | source_path：`STRING`- 工作区笔记本或目录的本地源路径。 堆栈配置模板文件的相对路径或文件系统中的绝对路径。 |
| jobs                 | [JobSettings](../api/latest/jobs.md#jobsjobsettings) 中的任何字段。 唯一一个 [JobSettings](../api/latest/jobs.md#jobsjobsettings) 中不需要，但堆栈 CLI 必需的字段：<br><br>name：`STRING`- 要部署的作业的名称。 为了不创建太多重复的作业，堆栈 CLI 在堆栈部署的作业中强制执行唯一名称。 | 无。                                                                                                                                                                            |
| dbfs                 | path：`STRING`- 匹配的远程 DBFS 路径。 必须以 `dbfs:/` 开头。 （例如： `dbfs:/this/is/a/sample/path`)<br><br>is_dir：`BOOL`- DBFS 路径是目录还是文件。                                                                                                                                                                                      | source_path：`STRING`- DBFS 文件或目录的本地源路径。 堆栈配置模板文件的相对路径或文件系统中的绝对路径。                 |

## <a name="resourceservice"></a>ResourceService

每个资源都属于与 Databricks REST API 相关的特定服务。
这些是堆栈 CLI 支持的服务。

| 服务          | 说明                                      |
|------------------|--------------------------------------------------|
| 工作区        | 工作区笔记本或目录。               |
| jobs             | Azure Databricks 作业。                         |
| dbfs             | DBFS 文件或目录。                        |

### <a name="stack-status-schema"></a><a id="stack-status-schema"> </a><a id="stackstatusschema"> </a>堆栈状态架构

## <a name="stackstatus"></a>StackStatus

堆栈状态文件是在使用 CLI 部署堆栈之后创建的。 顶级字段包括：

| 字段名称               | 类型                                      | 说明                                                                                                                                                          |
|--------------------------|-------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name                     | `STRING`                                  | 堆栈的名称。 此字段与 [StackConfig](#stackconfig) 中的字段相同。                                                                               |
| cli_version              | `STRING`                                  | 用于部署堆栈的 Databricks CLI 的版本。                                                                                                          |
| deployed_resources       | [ResourceStatus](#resourcestatus) 列表 | 每个已部署资源的状态。 对于在 [StackConfig](#stackconfig) 中定义的每个资源，此处都将生成相应的 [ResourceStatus](#resourcestatus)。 |

## <a name="resourcestatus"></a>ResourceStatus

| 字段名称               | 类型                                     | 说明                                                                                                |
|--------------------------|------------------------------------------|------------------------------------------------------------------------------------------------------------|
| id                       | `STRING`                                 | 资源的堆栈唯一 ID。                                                                        |
| 服务                  | [ResourceService](#resourceservice)      | 资源运行时所在的 REST API 服务。 `jobs`、<br>`workspace` 或 `dbfs` 中的一项。             |
| databricks_id            | [DatabricksId](#databricksid)            | 已部署资源的物理 ID。 实际架构取决于资源的类型（服务）。 |

## <a name="databricksid"></a>DatabricksId

一个 JSON 对象，其字段取决于服务。

| 服务          | JSON 中的字段        | 类型             | 描述                                                                                                                                                 |
|------------------|----------------------|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 工作区        | path                 | STRING           | Azure Databricks 工作区中笔记本或目录的绝对路径。 命名与[工作区 API](../api/latest/workspace.md) 一致。 |
| jobs             | job_id               | STRING           | 作业 ID，如 Azure Databricks 工作区中所示。 可用于更新已部署的作业。                                                     |
| dbfs             | path                 | STRING           | Azure Databricks 工作区中笔记本或目录的绝对路径。 命名与 [DBFS API](../api/latest/dbfs.md) 一致。           |