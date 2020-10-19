---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/01/2020
title: 库 API - Azure Databricks
description: 了解 Databricks 库 API。
ms.openlocfilehash: f0c003bc2912ed2f794f6897d0700c3149f4641a
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937674"
---
# <a name="libraries-api"></a>库 API

使用库 API 可以安装和卸载库，并获取群集上库的状态。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。

## <a name="all-cluster-statuses"></a><a id="all-cluster-statuses"> </a><a id="managedlibrariesmanagedlibraryserviceallclusterstatuses"> </a>所有群集状态

| 端点                                   | HTTP 方法     |
|--------------------------------------------|-----------------|
| `2.0/libraries/all-cluster-statuses`       | `GET`           |

获取所有群集上所有库的状态。 对于通过 API 或库 UI 安装在群集上的所有库，以及通过库 UI 设置为在所有群集上安装的库，可获得状态。 如果已将库设置为安装在所有群集上，则 `is_library_for_all_clusters` 将为 `true`，即使该库还安装在此特定群集上也是如此。

### <a name="example-response"></a>示例响应

```json
{
  "statuses": [
    {
      "cluster_id": "11203-my-cluster",
      "library_statuses": [
        {
          "library": {
            "jar": "dbfs:/mnt/libraries/library.jar"
          },
          "status": "INSTALLING",
          "messages": [],
          "is_library_for_all_clusters": false
        }
      ]
    },
    {
      "cluster_id": "20131-my-other-cluster",
      "library_statuses": [
        {
          "library": {
            "egg": "dbfs:/mnt/libraries/library.egg"
          },
          "status": "ERROR",
          "messages": ["Could not download library"],
          "is_library_for_all_clusters": false
        }
      ]
    }
  ]
}
```

### <a name="response-structure"></a><a id="managedlibrarieslistallclusterlibrarystatusesresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称     | 类型                                                                          | 描述                     |
|----------------|-------------------------------------------------------------------------------|---------------------------------|
| statuses       | 一个 [ClusterLibraryStatuses](#managedlibrariesclusterlibrarystatuses) 数组。 | 群集状态的列表。     |

## <a name="cluster-status"></a><a id="cluster-status"> </a><a id="managedlibrariesmanagedlibraryserviceclusterstatus"> </a>群集状态

| 端点                             | HTTP 方法     |
|--------------------------------------|-----------------|
| `2.0/libraries/cluster-status`       | `GET`           |

获取某个群集上库的状态。 对于通过 API 或库 UI 安装在此群集上的所有库，以及通过库 UI 设置为在所有群集上安装的库，可获得状态。 如果已将库设置为安装在所有群集上，则 `is_library_for_all_clusters` 将为 `true`，即使该库还安装在此群集上也是如此。

### <a name="example-request"></a>示例请求

```bash
/libraries/cluster-status?cluster_id=11203-my-cluster
```

### <a name="example-response"></a>示例响应

```json
{
  "cluster_id": "11203-my-cluster",
  "library_statuses": [
    {
      "library": {
        "jar": "dbfs:/mnt/libraries/library.jar"
      },
      "status": "INSTALLED",
      "messages": [],
      "is_library_for_all_clusters": false
    },
    {
      "library": {
        "pypi": {
          "package": "beautifulsoup4"
        },
      },
      "status": "INSTALLING",
      "messages": ["Successfully resolved package from PyPI"],
      "is_library_for_all_clusters": false
    },
    {
      "library": {
        "cran": {
          "package": "ada",
          "repo": "https://cran.us.r-project.org"
        },
      },
      "status": "FAILED",
      "messages": ["R package installation is not supported on this spark version.\nPlease upgrade to Runtime 3.2 or higher"],
      "is_library_for_all_clusters": false
    }
  ]
}
```

### <a name="request-structure"></a><a id="managedlibrariesclusterstatus"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                                |
|----------------|----------------|--------------------------------------------------------------------------------------------|
| cluster_id     | `STRING`       | 应检索其状态的群集的唯一标识符。 此字段为必需字段。 |

### <a name="response-structure"></a><a id="managedlibrariesclusterstatusresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称           | 类型                                                                | 描述                                 |
|----------------------|---------------------------------------------------------------------|---------------------------------------------|
| cluster_id           | `STRING`                                                            | 群集的唯一标识符。          |
| library_statuses     | 一个 [LibraryFullStatus](#managedlibrarieslibraryfullstatus) 数组 | 该群集上所有库的状态。     |

## <a name="install"></a><a id="install"> </a><a id="managedlibrariesmanagedlibraryserviceinstalllibraries"> </a>安装

| 端点                      | HTTP 方法     |
|-------------------------------|-----------------|
| `2.0/libraries/install`       | `POST`          |

在群集上安装库。 安装是异步的 - 它在请求出现之后在后台完成。

> [!IMPORTANT]
>
> 如果群集已终止，则此调用会失败。

在群集上安装 wheel 库就像直接在驱动程序和执行程序上对 wheel 文件运行 `pip` 命令一样。
将安装库 `setup.py` 文件中指定的所有依赖项，这需要库名称满足 [wheel 文件名约定](https://www.python.org/dev/peps/pep-0427/#file-name-convention)。

只有在启动新任务时，才会在执行程序上进行安装。
使用 Databricks Runtime 7.1 及更低版本时，库的安装顺序不确定。 对于 wheel 库，可以通过创建后缀为 `.wheelhouse.zip` 的 zip 文件（包括所有 wheel 文件）来确保安装顺序确定。

### <a name="example-request"></a>示例请求

```json
{
  "cluster_id": "10201-my-cluster",
  "libraries": [
    {
      "jar": "dbfs:/mnt/libraries/library.jar"
    },
    {
      "egg": "dbfs:/mnt/libraries/library.egg"
    },
    {
      "whl": "dbfs:/mnt/libraries/mlflow-0.0.1.dev0-py2-none-any.whl"
    },
    {
      "whl": "dbfs:/mnt/libraries/wheel-libraries.wheelhouse.zip"
    },
    {
      "maven": {
        "coordinates": "org.jsoup:jsoup:1.7.2",
        "exclusions": ["slf4j:slf4j"]
      }
    },
    {
      "pypi": {
        "package": "simplejson",
        "repo": "https://my-pypi-mirror.com"
      }
    },
    {
      "cran": {
        "package": "ada",
        "repo": "https://cran.us.r-project.org"
      }
    }
  ]
}
```

### <a name="request-structure"></a><a id="managedlibrariesinstalllibraries"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型                                            | 描述                                                                                    |
|----------------|-------------------------------------------------|------------------------------------------------------------------------------------------------|
| cluster_id     | `STRING`                                        | 要在其上安装这些库的群集的唯一标识符。 此字段为必需字段。 |
| 库      | 一个由[库](#managedlibrarieslibrary)构成的数组 | 要安装的库。                                                                      |

## <a name="uninstall"></a><a id="managedlibrariesmanagedlibraryserviceuninstalllibraries"> </a><a id="uninstall"> </a>卸载

| 端点                        | HTTP 方法     |
|---------------------------------|-----------------|
| `2.0/libraries/uninstall`       | `POST`          |

设置要在群集上卸载的库。 在重启群集之前，不会卸载这些库。 卸载未安装在群集上的库不会产生任何影响，不会出错。

### <a name="example-request"></a>示例请求

```json
{
  "cluster_id": "10201-my-cluster",
  "libraries": [
    {
      "jar": "dbfs:/mnt/libraries/library.jar"
    },
    {
      "cran": "ada"
    }
  ]
}
```

### <a name="request-structure"></a><a id="managedlibrariesuninstalllibraries"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型                                            | 描述                                                                                      |
|----------------|-------------------------------------------------|--------------------------------------------------------------------------------------------------|
| cluster_id     | `STRING`                                        | 要在其上卸载这些库的群集的唯一标识符。 此字段为必需字段。 |
| 库      | 一个由[库](#managedlibrarieslibrary)构成的数组 | 要卸载的库。                                                                      |

## <a name="data-structures"></a><a id="data-structures"> </a><a id="libraryadd"> </a>数据结构

### <a name="in-this-section"></a>本节内容：

* [ClusterLibraryStatuses](#clusterlibrarystatuses)
* [Library](#library)
* [LibraryFullStatus](#libraryfullstatus)
* [MavenLibrary](#mavenlibrary)
* [PythonPyPiLibrary](#pythonpypilibrary)
* [RCranLibrary](#rcranlibrary)
* [LibraryInstallStatus](#libraryinstallstatus)

### <a name="clusterlibrarystatuses"></a><a id="clusterlibrarystatuses"> </a><a id="managedlibrariesclusterlibrarystatuses"> </a>ClusterLibraryStatuses

| 字段名称           | 类型                                                                | 描述                                 |
|----------------------|---------------------------------------------------------------------|---------------------------------------------|
| cluster_id           | `STRING`                                                            | 群集的唯一标识符。          |
| library_statuses     | 一个 [LibraryFullStatus](#managedlibrarieslibraryfullstatus) 数组 | 该群集上所有库的状态。     |

### <a name="library"></a><a id="library"> </a><a id="managedlibrarieslibrary"> </a>库

| 字段名称                                 | 类型                                                                                                                                                                                          | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| jar、egg、whl、pypi、maven 或 cran | `STRING`、`STRING`、`STRING`、[PythonPyPiLibrary](#managedlibrariespythonpypilibrary)、[MavenLibrary](#managedlibrariesmavenlibrary) 或 [RCranLibrary](#managedlibrariesrcranlibrary) | 如果是 jar，则为要安装的 jar 的 DBFS URI。 例如：`{ "jar": "dbfs:/mnt/databricks/library.jar" }`。<br><br>如果是 egg，则为要安装的 egg 的 DBFS URI。 例如：`{ "egg": "dbfs:/my/egg" }`。<br><br>如果 whl，则为要安装的 wheel 或已压缩 wheel 的 URI。 仅支持 DBFS URI。 例如：`{ "whl": "dbfs:/my/whl" }`。<br><br>wheel 文件名需要使用[正确的约定](https://www.python.org/dev/peps/pep-0427/#file-format)。 如果要安装压缩的 wheel，文件名后缀应为 `.wheelhouse.zip`。<br><br>如果是 pypi，则为要安装的 PyPI 库的规范。 例如：<br>`{ "package": "simplejson" }`<br><br>如果是 maven，则为要安装的 Maven 库的规范。 例如：<br>`{ "coordinates": "org.jsoup:jsoup:1.7.2" }`<br><br>如果是 cran，则为要安装的 CRAN 库的规范。 |

### <a name="libraryfullstatus"></a><a id="libraryfullstatus"> </a><a id="managedlibrarieslibraryfullstatus"> </a>LibraryFullStatus

特定群集上的库的状态。

| 字段名称                      | 类型                                                          | 描述                                                                           |
|---------------------------------|---------------------------------------------------------------|---------------------------------------------------------------------------------------|
| 库                         | [Library](#managedlibrarieslibrary)                           | 库的唯一标识符。                                                    |
| status                          | [LibraryInstallStatus](#managedlibrarieslibraryinstallstatus) | 在群集上安装该库的状态。                                      |
| 计数                        | 一个由 `STRING` 构成的数组                                          | 迄今为止出现的针对此库的所有信息和警告消息。         |
| is_library_for_all_clusters     | `BOOL`                                                        | 是否已通过库 UI 将该库设置为在所有群集上安装。     |

### <a name="mavenlibrary"></a><a id="managedlibrariesmavenlibrary"> </a><a id="mavenlibrary"> </a>MavenLibrary

| 字段名称      | 类型                       | 描述                                                                                                                                                                                                                                                                                                                          |
|-----------------|----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 坐标     | `STRING`                   | Gradle 样式的 Maven 坐标。 例如：`org.jsoup:jsoup:1.7.2`。 此字段为必需字段。                                                                                                                                                                                                                                        |
| 存储库            | `STRING`                   | 要从中安装 Maven 包的 Maven 存储库。 如果省略此项，则同时搜索 Maven 中央存储库和 Spark 包。                                                                                                                                                                                                             |
| 排除      | 一个由 `STRING` 构成的数组       | 要排除的依赖项的列表。 例如：`["slf4j:slf4j", "*:hadoop-client"]`。<br><br>Maven 依赖项排除：[https://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html](https://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html)。 |

### <a name="pythonpypilibrary"></a><a id="managedlibrariespythonpypilibrary"> </a><a id="pythonpypilibrary"> </a>PythonPyPiLibrary

| 字段名称     | 类型           | 描述                                                                                                                                                                 |
|----------------|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 包        | `STRING`       | 要安装的 PyPI 包的名称。 另外还支持可选的确切版本规范。 示例：`simplejson` 和 `simplejson==3.8.0`。 此字段为必需字段。 |
| 存储库           | `STRING`       | 可在其中找到该包的存储库。 如果此项未指定，则使用默认的 pip 索引。                                                                             |

### <a name="rcranlibrary"></a><a id="managedlibrariesrcranlibrary"> </a><a id="rcranlibrary"> </a>RCranLibrary

| 字段名称     | 类型           | 描述                                                                                         |
|----------------|----------------|-----------------------------------------------------------------------------------------------------|
| 包        | `STRING`       | 要安装的 CRAN 包的名称。 此字段为必需字段。                                    |
| 存储库           | `STRING`       | 可在其中找到该包的存储库。 如果此项未指定，则使用默认的 CRAN 存储库。     |

### <a name="libraryinstallstatus"></a><a id="libraryinstallstatus"> </a><a id="managedlibrarieslibraryinstallstatus"> </a>LibraryInstallStatus

特定群集上某个库的状态。

| 状态                   | 描述                                                                                                                                                                        |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| PENDING                  | 尚未执行任何操作来安装该库。 此状态应该很短暂。                                                                                        |
| RESOLVING                | 正在从提供的存储库检索安装该库所需的元数据。<br><br>对于 Jar、Egg 和 Whl 库，此步骤不执行任何操作。                           |
| INSTALLING               | 正在通过向 Spark 添加资源或在 Spark 节点内执行系统命令来主动安装该库。                                                  |
| INSTALLED                | 已成功安装该库。                                                                                                                                       |
| SKIPPED                  | 由于 Scala 版本不兼容，已跳过 Databricks Runtime 7.0 或更高版本群集上的安装。                                                                        |
| FAILED                   | 安装中的某个步骤失败。 可在 messages 字段中找到详细信息。                                                                                             |
| UNINSTALL_ON_RESTART     | 该库已标记为要删除。 只有在重启群集时才能删除库，因此进入此状态的库会一直保留到群集重启为止。 |