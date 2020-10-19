---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/23/2020
title: DBFS API - Azure Databricks
description: 了解 Databricks DBFS API。
ms.openlocfilehash: f63a351ec6ac6070eb231285a7571b994380b8bc
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937682"
---
# <a name="dbfs-api"></a>DBFS API

DBFS API 是一种 Databricks API，可让你轻松地与各种数据源交互，而不必在每次读取文件时都提供凭据。 有关详细信息，请参阅 [Databricks 文件系统 (DBFS)](../../../data/databricks-file-system.md)。
有关 DBFS API 的易用命令行客户端，请参阅 [Databricks CLI](../../cli/index.md)。

> [!NOTE]
>
> 为了确保在负载较高的情况下也能提供高质量的服务，Azure Databricks 现在正针对 DBFS API 调用强制实施 API 速率限制。 限制按工作区设置，以确保公平使用和高可用性。 如果使用 Databricks CLI 0.12.0 及更高版本，可以进行自动重试。 建议所有客户切换到最新的 Databricks CLI 版本。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。

## <a name="add-block"></a><a id="add-block"> </a><a id="dbfsdbfsserviceaddblock"> </a>添加块

| 端点                   | HTTP 方法     |
|----------------------------|-----------------|
| `2.0/dbfs/add-block`       | `POST`          |

将数据块追加到由输入句柄指定的流。 如果该句柄不存在，则此调用会引发异常，并返回 `RESOURCE_DOES_NOT_EXIST`。 如果数据块超出 1 MB，则此调用会引发异常，并返回 `MAX_BLOCK_SIZE_EXCEEDED`。
请求示例：

```json
{
  "data": "ZGF0YWJyaWNrcwo=",
  "handle": 7904256
}
```

### <a name="request-structure"></a><a id="dbfsaddblock"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型          | 描述                                                                                        |
|----------------|---------------|----------------------------------------------------------------------------------------------------|
| 句柄         | `INT64`       | 打开的流上的句柄。 此字段为必需字段。                                              |
| 数据           | `BYTES`       | 要追加到流的 base64 编码数据。 此项的限制为 1 MB。 此字段为必需字段。 |

## <a name="close"></a><a id="close"> </a><a id="dbfsdbfsserviceclose"> </a>关闭

| 端点               | HTTP 方法     |
|------------------------|-----------------|
| `2.0/dbfs/close`       | `POST`          |

关闭由输入句柄指定的流。 如果该句柄不存在，则此调用会引发异常，并返回 `RESOURCE_DOES_NOT_EXIST`。

### <a name="request-structure"></a><a id="dbfsclose"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型          | 描述                                           |
|----------------|---------------|-------------------------------------------------------|
| 句柄         | `INT64`       | 打开的流上的句柄。 此字段为必需字段。 |

## <a name="create"></a><a id="create"> </a><a id="dbfsdbfsservicecreate"> </a>创建

| 端点                | HTTP 方法     |
|-------------------------|-----------------|
| `2.0/dbfs/create`       | `POST`          |

打开流以将内容写入文件，并返回此流的句柄。 此句柄上有一个 10 分钟的空闲超时。 如果文件或目录已存在于给定路径中，并且 overwrite 设置为 false，则此调用会引发异常，并返回 `RESOURCE_ALREADY_EXISTS`。 文件上传的典型工作流将如下所述：

1. 发出 `create` 调用并获取句柄。
2. 使用你有的句柄发出一个或多个 `add-block` 调用。
3. 使用你有的句柄发出一个 `close` 调用。

### <a name="request-structure"></a><a id="dbfscreate"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                                                        |
|----------------|----------------|--------------------------------------------------------------------------------------------------------------------|
| path           | `STRING`       | 新文件的路径。 路径应为绝对 DBFS 路径（例如 `/mnt/foo.txt`）。 此字段为必需字段。 |
| overwrite      | `BOOL`         | 一个标志，用于指定是否覆盖现有文件。                                               |

### <a name="response-structure"></a><a id="dbfscreateresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称     | 类型          | 描述                                                                                                           |
|----------------|---------------|-----------------------------------------------------------------------------------------------------------------------|
| 句柄         | `INT64`       | 一个句柄，该句柄随后应该传递到 AddBlock，并在通过流写入到文件时关闭调用。 |

## <a name="delete"></a><a id="dbfsdbfsservicedelete"> </a><a id="delete"> </a>删除

| 端点                | HTTP 方法     |
|-------------------------|-----------------|
| `2.0/dbfs/delete`       | `POST`          |

删除文件或目录（可以选择以递归方式删除目录中的所有文件）。 如果路径为非空目录且 recursive 设置为 false，或出现其他类似错误，则此调用会引发异常，并返回 `IO_ERROR`。

删除大量文件时，删除操作以增量方式执行。
此调用在大约 45 秒后返回响应，并出现一条错误消息（503 服务不可用），要求你重新调用删除操作，直至完全删除目录结构。 例如：

```json
{
  "error_code":"PARTIAL_DELETE","message":"The requested operation has deleted 324 files. There are more files remaining. You must make another request to delete more."
}
```

对于删除 1 万个以上文件的操作，我们建议不要使用 DBFS REST API，而是使用[文件系统实用工具](../../databricks-utils.md#dbutils-fs)在群集上下文中执行此类操作。 `dbutils.fs` 涵盖 DBFS REST API 的功能范围，但仅限在笔记本内部。 使用笔记本运行此类操作可提供更好的控制和可管理性（例如，选择性删除），并可自动执行定期的删除作业。

### <a name="request-structure"></a><a id="dbfsdelete"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                                                                                 |
|----------------|----------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| path           | `STRING`       | 要删除的文件或目录的路径。 路径应为绝对 DBFS 路径（例如 `/mnt/foo/`）。 此字段为必需字段。          |
| recursive      | `BOOL`         | 是否以递归方式删除目录的内容。 无需提供递归标志即可删除空目录。 |

## <a name="get-status"></a><a id="dbfsdbfsservicegetstatus"> </a><a id="get-status"> </a>获取状态

| 端点                    | HTTP 方法     |
|-----------------------------|-----------------|
| `2.0/dbfs/get-status`       | `GET`           |

获取文件或目录的文件信息。 如果该文件或目录不存在，则此调用会引发异常，并返回 `RESOURCE_DOES_NOT_EXIST`。

### <a name="request-structure"></a><a id="dbfsgetstatus"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                                                              |
|----------------|----------------|--------------------------------------------------------------------------------------------------------------------------|
| path           | `STRING`       | 文件或目录的路径。 路径应为绝对 DBFS 路径（例如 `/mnt/foo/`）。 此字段为必需字段。 |

### <a name="response-structure"></a><a id="dbfsgetstatusresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称     | 类型           | 描述                                                             |
|----------------|----------------|-------------------------------------------------------------------------|
| path           | `STRING`       | 文件或目录的路径。                                      |
| is_dir         | `BOOL`         | 如果路径是目录，则此项的值为 true。                                        |
| file_size      | `INT64`        | 文件的长度（以字节为单位）；如果路径是目录，则此项的值为零。     |

## <a name="list"></a><a id="dbfsdbfsservicelist"> </a><a id="list"> </a>列出

| 端点              | HTTP 方法     |
|-----------------------|-----------------|
| `2.0/dbfs/list`       | `GET`           |

列出目录的内容或文件的详细信息。 如果该文件或目录不存在，则此调用会引发异常，并返回 `RESOURCE_DOES_NOT_EXIST`。

在大型目录中调用 `list` 时，`list` 操作会在大约 60 秒后超时。 强烈建议仅在包含的文件数小于 1 万的目录上使用 `list`，不要将 DBFS REST API 用于执行会列出 1 万个以上文件的操作。 我们建议你使用[文件系统实用工具](../../databricks-utils.md#dbutils-fs)在群集上下文中执行此类操作，该实用工具提供相同的功能，但没有超时。

回复示例：

```json
{
  "files": [
    {
      "path": "/a.cpp",
      "is_dir": false,
      "file_size": 261
    },
    {
      "path": "/databricks-results",
      "is_dir": true,
      "file_size": 0
    }
  ]
}
```

### <a name="request-structure"></a><a id="dbfsliststatus"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                                                              |
|----------------|----------------|--------------------------------------------------------------------------------------------------------------------------|
| path           | `STRING`       | 文件或目录的路径。 路径应为绝对 DBFS 路径（例如 `/mnt/foo/`）。 此字段为必需字段。 |

### <a name="response-structure"></a><a id="dbfsliststatusresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称     | 类型                                  | 描述                                                                              |
|----------------|---------------------------------------|------------------------------------------------------------------------------------------|
| 文件          | [FileInfo](#dbfsfileinfo) 的数组 | FileInfo 列表，用于描述目录或文件的内容。                          |

## <a name="mkdirs"></a><a id="dbfsdbfsservicemkdirs"> </a><a id="mkdirs"> </a>Mkdirs

| 端点                | HTTP 方法     |
|-------------------------|-----------------|
| `2.0/dbfs/mkdirs`       | `POST`          |

创建给定目录和必要的父目录（如果不存在）。 如果在输入路径的任何前缀处存在一个文件（而不是目录），则此调用会引发异常，并返回 `RESOURCE_ALREADY_EXISTS`。 如果此操作失败，则可能已成功创建了一些必需的父目录。

### <a name="request-structure"></a><a id="dbfsmkdirs"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                                                          |
|----------------|----------------|----------------------------------------------------------------------------------------------------------------------|
| path           | `STRING`       | 新目录的路径。 路径应为绝对 DBFS 路径（例如 `/mnt/foo/`）。 此字段为必需字段。 |

## <a name="move"></a><a id="dbfsdbfsservicemove"> </a><a id="move"> </a>移动

| 端点              | HTTP 方法     |
|-----------------------|-----------------|
| `2.0/dbfs/move`       | `POST`          |

在 DBFS 中将文件从一个位置移到另一个位置。 如果源文件不存在，则此调用会引发异常，并返回 `RESOURCE_DOES_NOT_EXIST`。 如果目标路径中已经存在一个文件，则此调用会引发异常，并返回 `RESOURCE_ALREADY_EXISTS`。 如果给定的源路径是一个目录，则此调用始终会以递归方式移动所有文件。

移动大量文件时，API 调用会在大约 60 秒后超时，这可能会导致只有一部分数据被移动。 因此，对于移动 1 万个以上文件的操作，我们强烈建议不要使用 DBFS REST API。 我们建议你使用笔记本的[文件系统实用工具](../../databricks-utils.md#dbutils-fs)在群集上下文中执行此类操作，该实用工具提供相同的功能，但没有超时。

### <a name="request-structure"></a><a id="dbfsmove"> </a><a id="request-structure"> </a>请求结构

| 字段名称           | 类型           | 描述                                                                                                                          |
|----------------------|----------------|--------------------------------------------------------------------------------------------------------------------------------------|
| source_path          | `STRING`       | 文件或目录的源路径。 路径应为绝对 DBFS 路径（例如 `/mnt/foo/`）。 此字段为必需字段。      |
| destination_path     | `STRING`       | 文件或目录的目标路径。 路径应为绝对 DBFS 路径（例如 `/mnt/bar/`）。 此字段为必需字段。 |

## <a name="put"></a><a id="dbfsdbfsserviceput"> </a><a id="put"> </a>放置

| 端点             | HTTP 方法     |
|----------------------|-----------------|
| `2.0/dbfs/put`       | `POST`          |

通过使用“多部分表单 POST”来上传文件。 它主要用于流式上传，但也可用作方便的单个调用来上传数据。 用法示例：

在以下示例中，请将 `<databricks-instance>` 替换为 Azure Databricks 部署的[工作区 URL](../../../workspace/workspace-details.md#workspace-url)。

```bash
curl -F contents=@localsrc -F path="PATH" https://<databricks-instance>/api/2.0/dbfs/put
```

`localsrc` 是要上传的本地文件的路径，并且只有“多部分表单 POST”（即，将 `-F `` or ``--form` 与 `curl` 结合使用）支持这种用法。

也可将内容作为 base64 字符串传递。 示例：

```bash
curl -F contents="BASE64" -F path="PATH" https://<databricks-instance>/api/2.0/dbfs/put
```

```bash
curl  -H "Content-Type: application/json" -d '{"path":"PATH","contents":"BASE64"}' https://<databricks-instance>/api/2.0/dbfs/put``
```

可以使用 `contents`（即非流式处理）参数传递的数据量限制为 1 MB；如果超出，则会引发 `MAX_BLOCK_SIZE_EXCEEDED`。 如果要上传大文件，请使用流式上传。 有关详细信息，请参阅[创建](#dbfsdbfsservicecreate)、[添加块](#dbfsdbfsserviceaddblock)和[关闭](#dbfsdbfsserviceclose)。

### <a name="request-structure"></a><a id="dbfsput"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                                                     |
|----------------|----------------|-----------------------------------------------------------------------------------------------------------------|
| path           | `STRING`       | 新文件的路径。 路径应为绝对 DBFS 路径（例如 `/mnt/foo/`）。 此字段为必需字段。 |
| 内容       | `BYTES`        | 此参数可能不存在，将会改用已发布的文件。                                         |
| overwrite      | `BOOL`         | 一个标志，用于指定是否覆盖现有文件。                                                    |

## <a name="read"></a><a id="dbfsdbfsserviceread"> </a><a id="read"> </a>读取

| 端点              | HTTP 方法     |
|-----------------------|-----------------|
| `2.0/dbfs/read`       | `GET`           |

返回文件的内容。 如果文件不存在，则此调用会引发异常，并返回 `RESOURCE_DOES_NOT_EXIST`。 如果路径是目录，则读取长度为负数；如果偏移量为负，则此调用会引发异常，并返回 `INVALID_PARAMETER_VALUE`。 如果读取长度超出 1 MB，则此调用会引发异常，并返回 `MAX_READ_SIZE_EXCEEDED`。 如果 `offset + length` 超出文件中的字节数，则读取内容，直到文件结尾。

### <a name="request-structure"></a><a id="dbfsread"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                                                         |
|----------------|----------------|---------------------------------------------------------------------------------------------------------------------|
| path           | `STRING`       | 要读取的文件的路径。 路径应为绝对 DBFS 路径（例如 `/mnt/foo/`）。 此字段为必需字段。 |
| offset         | `INT64`        | 要从其开始读取的偏移量（以字节为单位）。                                                                                   |
| length         | `INT64`        | 要从该偏移量开始读取的字节数。 其限制为 1 MB，默认值为 0.5 MB。      |

### <a name="response-structure"></a><a id="dbfsreadresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称     | 类型          | 描述                                                                                                                                                             |
|----------------|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| bytes_read     | `INT64`       | 读取的字节数（可能会小于 length，如果遇到文件结尾的话）。 这是指在未编码的版本中读取的字节数（响应数据采用 base64 编码）。 |
| 数据           | `BYTES`       | 读取的文件的 base64 编码内容。                                                                                                                           |

## <a name="data-structures"></a><a id="data-structures"> </a><a id="dbfsadd"> </a>数据结构

### <a name="in-this-section"></a>本节内容：

* [FileInfo](#fileinfo)

### <a name="fileinfo"></a><a id="dbfsfileinfo"> </a><a id="fileinfo"> </a>FileInfo

存储文件或目录的属性。

| 字段名称     | 类型           | 描述                                                             |
|----------------|----------------|-------------------------------------------------------------------------|
| path           | `STRING`       | 文件或目录的路径。                                      |
| is_dir         | `BOOL`         | 如果路径是目录，则此项的值为 true。                                        |
| file_size      | `INT64`        | 文件的长度（以字节为单位）；如果路径是目录，则此项的值为零。     |