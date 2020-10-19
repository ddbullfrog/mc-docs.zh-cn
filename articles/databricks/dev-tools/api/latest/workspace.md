---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: 工作区 API - Azure Databricks
description: 了解 Databricks 工作区 API。
ms.openlocfilehash: d4100499a3ff4bb3156f2c8520801b835ee7db75
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937659"
---
# <a name="workspace-api"></a>工作区 API

使用工作区 API 可列出、导入、导出和删除笔记本和文件夹。 对工作区 API 的请求的最大允许大小为 10MB。 有关此 API 的操作方法指南，请参阅[工作区示例](examples.md#workspace-api-example)。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。

## <a name="delete"></a><a id="delete"> </a><a id="workspaceworkspaceservicedelete"> </a>删除

| 端点                     | HTTP 方法     |
|------------------------------|-----------------|
| `2.0/workspace/delete`       | `POST`          |

删除对象或目录（可以选择以递归方式删除目录中的所有对象）。
如果 `path` 不存在，则此调用会返回错误 `RESOURCE_DOES_NOT_EXIST`。
如果 `path` 为非空目录且 `recursive` 设置为 `false`，则此调用会返回错误 `DIRECTORY_NOT_EMPTY`。
对象删除无法撤消，且以递归方式删除目录不是原子操作。
请求的示例：

```json
{
  "path": "/Users/user@example.com/project",
  "recursive": true
}
```

### <a name="request-structure"></a><a id="request-structure"> </a><a id="workspacedelete"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                                                                                                                                                                                         |
|----------------|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| path           | `STRING`       | 笔记本或目录的绝对路径。 此字段为必需字段。                                                                                                                                                                             |
| recursive      | `BOOL`         | 一个标志，用于指定是否以递归方式删除对象。 默认情况下，它为 `false`。 请注意，此删除目录操作不是原子操作。 如果此操作中途失败，则该目录下的某些对象可能会被删除，并且无法撤消。 |

## <a name="export"></a><a id="export"> </a><a id="workspaceworkspaceserviceexport"> </a>导出

| 端点                     | HTTP 方法     |
|------------------------------|-----------------|
| `2.0/workspace/export`       | `GET`           |

导出笔记本或整个目录的内容。
如果 `path` 不存在，则此调用会返回错误 `RESOURCE_DOES_NOT_EXIST`。
只能以 `DBC` 格式导出目录。
如果导出的数据超过大小限制，则此调用会返回错误 `MAX_NOTEBOOK_SIZE_EXCEEDED`。
此 API 不支持导出库。
请求的示例：

```json
{
  "path": "/Users/user@example.com/project/ScalaExampleNotebook",
  "format": "SOURCE"
}
```

响应的示例，其中 `content` 已进行 base64 编码：

```json
{
  "content": "Ly8gRGF0YWJyaWNrcyBub3RlYm9vayBzb3VyY2UKMSsx",
}
```

也可通过启用 `direct_download` 来下载导出的文件：

```bash
curl -n -o example.scala \
  'https://<databricks-instance>/api/2.0/workspace/export?path=/Users/user@example.com/ScalaExampleNotebook&direct_download=true'
```

在以下示例中，请将 `<databricks-instance>` 替换为 Azure Databricks 部署的[工作区 URL](../../../workspace/workspace-details.md#workspace-url)。

### <a name="request-structure"></a><a id="request-structure"> </a><a id="workspaceexport"> </a>请求结构

| 字段名称          | 类型                                  | 描述                                                                                                                                                                                                                                                                              |
|---------------------|---------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| path                | `STRING`                              | 笔记本或目录的绝对路径。 仅 `DBC` 支持导出目录。 此字段为必需字段。                                                                                                                                                               |
| format              | [ExportFormat](#notebookexportformat) | 此项指定已导出文件的格式。 默认情况下，该属性为 `SOURCE`。 但是，它可以是以下值之一：<br>`SOURCE`, `HTML`, `JUPYTER`, `DBC`. 值区分大小写。                                                                                                              |
| direct_download     | `BOOL`                                | 一个标志，用于启用直接下载。 如果它为 `true`，则响应将是导出的文件本身。 否则，响应会包含 base64 编码字符串形式的内容。 请参阅[导出笔记本或文件夹](examples.md#workspace-api-export-example)，详细了解如何使用它。 |

### <a name="response-structure"></a><a id="response-structure"> </a><a id="workspaceexportresponse"> </a>响应结构

| 字段名称     | 类型          | 描述                                                                                                                    |
|----------------|---------------|--------------------------------------------------------------------------------------------------------------------------------|
| 内容        | `BYTES`       | Base64 编码的内容。 如果超过限制 (10MB)，则会引发错误代码为 `MAX_NOTEBOOK_SIZE_EXCEEDED` 的异常。 |

## <a name="get-status"></a><a id="get-status"> </a><a id="workspaceworkspaceservicegetstatus"> </a>获取状态

| 端点                         | HTTP 方法     |
|----------------------------------|-----------------|
| `2.0/workspace/get-status`       | `GET`           |

获取对象或目录的状态。
如果 `path` 不存在，则此调用会返回错误 `RESOURCE_DOES_NOT_EXIST`。
请求的示例：

```json
{
  "path": "/Users/user@example.com/project/ScaleExampleNotebook"
}
```

响应的示例：

```json
{
  "path": "/Users/user@example.com/project/ScalaExampleNotebook",
  "language": "SCALA",
  "object_type": "NOTEBOOK",
  "object_id": 789
}
```

### <a name="request-structure"></a><a id="request-structure"> </a><a id="workspacegetstatus"> </a>请求结构

| 字段名称     | 类型           | 描述                                                             |
|----------------|----------------|-------------------------------------------------------------------------|
| path           | `STRING`       | 笔记本或目录的绝对路径。 此字段为必需字段。 |

### <a name="response-structure"></a><a id="response-structure"> </a><a id="workspacegetstatusresponse"> </a>响应结构

| 字段名称      | 类型                               | 说明                                                                                |
|-----------------|------------------------------------|--------------------------------------------------------------------------------------------|
| object_type     | [ObjectType](#workspaceobjecttype) | 对象的类型。 它可以是 `NOTEBOOK`、`DIRECTORY` 或 `LIBRARY`。                  |
| object_id       | `INT64`                            | `NOTEBOOK` 或 `DIRECTORY` 的唯一标识符。                                         |
| path            | `STRING`                           | 对象的绝对路径。                                                           |
| 语言        | [语言](#notebooklanguage)      | 对象的语言。 仅当对象类型为 `NOTEBOOK` 时才设置此值。       |

## <a name="import"></a><a id="import"> </a><a id="workspaceworkspaceserviceimport"> </a>导入

| 端点                     | HTTP 方法     |
|------------------------------|-----------------|
| `2.0/workspace/import`       | `POST`          |

导入笔记本或整个目录的内容。
如果 `path` 已存在并且 `overwrite` 设置为 `false`，则此调用会返回错误 `RESOURCE_ALREADY_EXISTS`。
只能使用 `DBC` 格式来导入目录。
请求的示例，其中 `content` 是 `1+1` 的 base64 编码字符串：

```json
{
  "content": "MSsx\n",
  "path": "/Users/user@example.com/project/ScalaExampleNotebook",
  "language": "SCALA",
  "overwrite": true,
  "format": "SOURCE"
}
```

也可直接导入本地文件。

在以下示例中，请将 `<databricks-instance>` 替换为 Azure Databricks 部署的[工作区 URL](../../../workspace/workspace-details.md#workspace-url)。

```shell
curl -n -F path=/Users/user@example.com/project/ScalaExampleNotebook -F language=SCALA \
  -F content=@example.scala \
  https://<databricks-instance>/api/2.0/workspace/import
```

### <a name="request-structure"></a><a id="request-structure"> </a><a id="workspaceimport"> </a>请求结构

| 字段名称     | 类型                                  | 描述                                                                                                                                                                                                                                                                                                                                                |
|----------------|---------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| path           | `STRING`                              | 笔记本或目录的绝对路径。 仅 `DBC` 格式支持导入目录。 此字段为必需字段。                                                                                                                                                                                                                              |
| format         | [ExportFormat](#notebookexportformat) | 此项指定要导入的文件的格式。 默认情况下，该属性为 `SOURCE`。 但是，它可以是以下值之一：<br>`SOURCE`, `HTML`, `JUPYTER`, `DBC`. 值区分大小写。                                                                                                                                                                          |
| 语言       | [语言](#notebooklanguage)         | 语言。 如果格式设置为 `SOURCE`，则此字段为必填字段；否则，它会被系统忽略。                                                                                                                                                                                                                                                         |
| 内容        | `BYTES`                               | Base64 编码的内容。 此项的限制为 10 MB。 如果超过限制 (10MB)，则会引发错误代码为 `MAX_NOTEBOOK_SIZE_EXCEEDED` 的异常。 此参数可能不存在，将会改用已发布的文件。 请参阅[导入笔记本或目录](examples.md#workspace-api-import-example)，详细了解如何使用它。 |
| overwrite      | `BOOL`                                | 一个标志，用于指定是否覆盖现有对象。 默认情况下，它为 `false`。 对于 `DBC` 格式，不支持覆盖，因为它可能包含目录。                                                                                                                                                                                     |

## <a name="list"></a><a id="list"> </a><a id="workspaceworkspaceservicelist"> </a>列表

| 端点                   | HTTP 方法     |
|----------------------------|-----------------|
| `2.0/workspace/list`       | `GET`           |

列出目录的内容，如果不是目录，则列出对象。
如果输入路径不存在，则此调用会返回错误 `RESOURCE_DOES_NOT_EXIST`。
请求的示例：

```json
{
  "path": "/Users/user@example.com/"
}
```

响应的示例：

```json
{
  "objects": [
    {
      "path": "/Users/user@example.com/project",
      "object_type": "DIRECTORY",
      "object_id": 123
    },
    {
      "path": "/Users/user@example.com/PythonExampleNotebook",
      "language": "PYTHON",
      "object_type": "NOTEBOOK",
      "object_id": 456
    }
  ]
}
```

### <a name="request-structure"></a><a id="request-structure"> </a><a id="workspacelist"> </a>请求结构

| 字段名称     | 类型           | 描述                                                             |
|----------------|----------------|-------------------------------------------------------------------------|
| path           | `STRING`       | 笔记本或目录的绝对路径。 此字段为必需字段。 |

### <a name="response-structure"></a><a id="response-structure"> </a><a id="workspacelistresponse"> </a>响应结构

| 字段名称     | 类型                                           | 描述          |
|----------------|------------------------------------------------|----------------------|
| 对象        | 一个 [ObjectInfo](#workspaceobjectinfo) 数组 | 对象的列表。     |

## <a name="mkdirs"></a><a id="mkdirs"> </a><a id="workspaceworkspaceservicemkdirs"> </a>Mkdirs

| 端点                     | HTTP 方法     |
|------------------------------|-----------------|
| `2.0/workspace/mkdirs`       | `POST`          |

创建给定目录和必要的父目录（如果不存在）。
如果在输入路径的任何前缀处存在一个对象（而不是目录），则此调用会返回错误 `RESOURCE_ALREADY_EXISTS`。
如果此操作失败，可能已成功创建了一些必需的父目录。
请求的示例：

```json
{
  "path": "/Users/user@example.com/project"
}
```

### <a name="request-structure"></a><a id="request-structure"> </a><a id="workspacemkdirs"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                                                                                                                                              |
|----------------|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| path           | `STRING`       | 目录的绝对路径。 如果父目录不存在，则也会创建它们。 如果目录已存在，此命令不需执行任何操作即可成功执行。 此字段为必需字段。 |

## <a name="data-structures"></a><a id="data-structures"> </a><a id="workspaceadd"> </a>数据结构

### <a name="in-this-section"></a>本节内容：

* [ObjectInfo](#objectinfo)
* [ExportFormat](#exportformat)
* [语言](#language)
* [ObjectType](#objecttype)

### <a name="objectinfo"></a><a id="objectinfo"> </a><a id="workspaceobjectinfo"> </a>ObjectInfo

工作区中对象的信息。 它由 `list` 和 `get-status` 返回。

| 字段名称      | 类型                               | 说明                                                                                |
|-----------------|------------------------------------|--------------------------------------------------------------------------------------------|
| object_type     | [ObjectType](#workspaceobjecttype) | 对象的类型。 它可以是 `NOTEBOOK`、`DIRECTORY` 或 `LIBRARY`。                  |
| object_id       | `INT64`                            | `NOTEBOOK` 或 `DIRECTORY` 的唯一标识符。                                         |
| path            | `STRING`                           | 对象的绝对路径。                                                           |
| 语言        | [语言](#notebooklanguage)      | 对象的语言。 仅当对象类型为 `NOTEBOOK` 时才设置此值。       |

### <a name="exportformat"></a><a id="exportformat"> </a><a id="notebookexportformat"> </a>ExportFormat

笔记本导入和导出的格式。

| 格式      | 描述                                                                    |
|-------------|--------------------------------------------------------------------------------|
| 源      | 笔记本将以源代码的形式导入/导出。                         |
| HTML        | 笔记本将以 HTML 文件的形式导入/导出。                        |
| JUPYTER     | 笔记本将以 Jupyter/IPython Notebook 文件的形式导入/导出。     |
| DBC         | 笔记本将以 Databricks 存档格式导入/导出。           |

### <a name="language"></a><a id="language"> </a><a id="notebooklanguage"> </a>语言

笔记本的语言。

| 语言     | 说明          |
|--------------|----------------------|
| SCALA        | Scala 笔记本。      |
| PYTHON       | Python 笔记本。     |
| SQL          | SQL 笔记本。        |
| R            | R 笔记本。          |

### <a name="objecttype"></a><a id="objecttype"> </a><a id="workspaceobjecttype"> </a>ObjectType

工作区中对象的类型。

| 类型          | 描述     |
|---------------|-----------------|
| NOTEBOOK      | 笔记本        |
| 目录     | Directory       |
| LIBRARY       | 库         |