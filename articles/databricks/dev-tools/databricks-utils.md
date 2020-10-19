---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: Databricks 实用工具 - Azure Databricks
description: 了解和学习如何使用 Databricks 实用工具。
ms.openlocfilehash: 1de87646bbba73ee5196a19a6a8df872df996579
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937634"
---
# <a name="databricks-utilities"></a>Databricks 实用工具

使用 Databricks 实用工具 (DBUtils) 可以轻松地执行各种强大的任务组合。 可以使用这些实用工具来有效地处理对象存储，对笔记本进行链接和参数化，以及处理机密。 对 DBUtils 的支持仅限于笔记本内。

所有 `dbutils` 实用工具均在 Python、R 和 Scala 笔记本中提供。 [文件系统实用工具](#dbutils-fs)在 R 笔记本中不可用；但是，可以使用[语言 magic 命令](../notebooks/notebooks-use.md#language-magic)在 R 和 SQL 笔记本中调用这些 `dbutils` 方法。 例如，若要列出 R 或 SQL 笔记本中的 [Azure Databricks 数据集](../data/databricks-datasets.md#databricks-datasets) DBFS 文件夹，请运行以下命令：

```python
dbutils.fs.ls("/databricks-datasets")
```

也可使用 `%fs`：

```bash
%fs ls /databricks-datasets
```

## <a name="file-system-utilities"></a><a id="dbutils-fs"> </a><a id="file-system-utilities"> </a>文件系统实用工具

文件系统实用工具会访问 [Databricks 文件系统 (DBFS)](../data/databricks-file-system.md)，让你可以更轻松地将 Azure Databricks 用作文件系统。 有关详细信息，请运行：

```python
dbutils.fs.help()
```

```
cp(from: String, to: String, recurse: boolean = false): boolean -> Copies a file or directory, possibly across FileSystems
head(file: String, maxBytes: int = 65536): String -> Returns up to the first 'maxBytes' bytes of the given file as a String encoded in UTF-8
ls(dir: String): Seq -> Lists the contents of a directory
mkdirs(dir: String): boolean -> Creates the given directory if it does not exist, also creating any necessary parent directories
mv(from: String, to: String, recurse: boolean = false): boolean -> Moves a file or directory, possibly across FileSystems
put(file: String, contents: String, overwrite: boolean = false): boolean -> Writes the given String out to a file, encoded in UTF-8
rm(dir: String, recurse: boolean = false): boolean -> Removes a file or directory

mount(source: String, mountPoint: String, encryptionType: String = "", owner: String = null, extraConfigs: Map = Map.empty[String, String]): boolean -> Mounts the given source directory into DBFS at the given mount point
mounts: Seq -> Displays information about what is mounted within DBFS
refreshMounts: boolean -> Forces all machines in this cluster to refresh their mount cache, ensuring they receive the most recent information
unmount(mountPoint: String): boolean -> Deletes a DBFS mount point
```

### <a name="dbutilsfsls-command"></a>dbutils.fs.ls 命令

`ls` 命令返回的序列包含以下属性：

| Attribute      | 类型            | 描述                                                             |
|----------------|-----------------|-------------------------------------------------------------------------|
| path           | string          | 文件或目录的路径。                                      |
| name           | 字符串          | 文件或目录的名称。                                      |
| isDir()        | boolean         | 如果路径是目录，则此项为 true。                                        |
| 大小           | long/int64      | 文件的长度（以字节为单位）；如果路径是目录，则此项为零。     |

> [!NOTE]
>
> 可以使用 `help` 获取有关每个命令的详细信息，例如 `dbutils.fs.help("ls")`

## <a name="notebook-workflow-utilities"></a><a id="dbutils-workflow"> </a><a id="notebook-workflow-utilities"> </a>笔记本工作流实用工具

可以通过笔记本工作流将笔记本链接在一起，并对其结果进行操作。 请参阅[笔记本工作流](../notebooks/notebook-workflows.md)。 有关详细信息，请运行：

```python
dbutils.notebook.help()
```

```
exit(value: String): void -> This method lets you exit a notebook with a value
run(path: String, timeoutSeconds: int, arguments: Map): String -> This method runs a notebook and returns its exit value.
```

> [!NOTE]
>
> 从 `run` 返回的字符串值的最大长度为 5 MB。 请参阅[运行获取输出](api/latest/jobs.md#jobsjobsservicegetrunoutput)。

> [!NOTE]
>
> 可以使用 `help` 获取有关每个命令的详细信息，例如 `dbutils.notebook.help("exit")`

## <a name="widget-utilities"></a><a id="dbutils-widgets"> </a><a id="widget-utilities"> </a>小组件实用工具

可以通过小组件将笔记本参数化。 请参阅[小组件](../notebooks/widgets.md)。 有关详细信息，请运行：

```python
dbutils.widgets.help()
```

```
combobox(name: String, defaultValue: String, choices: Seq, label: String): void -> Creates a combobox input widget with a given name, default value and choices
dropdown(name: String, defaultValue: String, choices: Seq, label: String): void -> Creates a dropdown input widget a with given name, default value and choices
get(name: String): String -> Retrieves current value of an input widget
getArgument(name: String, optional: String): String -> (DEPRECATED) Equivalent to get
multiselect(name: String, defaultValue: String, choices: Seq, label: String): void -> Creates a multiselect input widget with a given name, default value and choices
remove(name: String): void -> Removes an input widget from the notebook
removeAll: void -> Removes all widgets in the notebook
text(name: String, defaultValue: String, label: String): void -> Creates a text input widget with a given name and default value
```

> [!NOTE]
>
> 可以使用 `help` 获取有关每个命令的详细信息，例如 `dbutils.widgets.help("combobox")`

## <a name="secrets-utilities"></a><a id="dbutils-secrets"> </a><a id="secrets-utilities"> </a>机密实用工具

可以利用机密存储和访问敏感的凭据信息，而无需使其在笔记本中可见。 请参阅[机密管理](../security/secrets/index.md#secrets-user-guide)和[在笔记本中使用机密](../security/secrets/example-secret-workflow.md#secret-example-notebook)。 有关详细信息，请运行：

```python
dbutils.secrets.help()
```

```
get(scope: String, key: String): String -> Gets the string representation of a secret value with scope and key
getBytes(scope: String, key: String): byte[] -> Gets the bytes representation of a secret value with scope and key
list(scope: String): Seq -> Lists secret metadata for secrets within a scope
listScopes: Seq -> Lists secret scopes
```

> [!NOTE]
>
> 可以使用 `help` 获取有关每个命令的详细信息，例如 `dbutils.secrets.help("get")`

## <a name="library-utilities"></a><a id="dbutils-library"> </a><a id="library-utilities"> </a>库实用工具

可以通过库实用工具安装 Python 库并创建作用域为笔记本会话的环境。 这些库在驱动程序和执行程序上均可用，因此你可以在 UDF 中引用它们。 这样做可以实现以下操作：

* 在笔记本自身中组织笔记本的库依赖项。
* 具有不同库依赖项的笔记本用户共享群集而不受干扰。

分离笔记本会破坏此环境。 但是，你可以重新创建它，只需在笔记本中重新运行库 `install` API 命令即可。 请参阅 `restartPython` API，了解如何才能重置笔记本状态而不失去环境。

> [!NOTE]
>
> 库实用工具在 Databricks Runtime ML 或用于基因组学的 Databricks Runtime 上不可用。 请改为参阅[作用域为笔记本的 Python 库](../libraries/notebooks-python-libraries.md)。
>
> 对于 Databricks Runtime 7.1 或更高版本，还可以使用 `%pip` magic 命令安装作用域为笔记本的库。 请参阅[作用域为笔记本的 Python 库](../libraries/notebooks-python-libraries.md)。

默认情况下，库实用工具处于启用状态。 因此，默认情况下会通过使用单独的 Python 可执行文件隔离每个笔记本的 Python 环境，该可执行文件在笔记本附加到群集并继承群集上的默认 Python 环境时创建。 通过 [init script](../clusters/init-scripts.md) 安装到 Azure Databricks Python 环境中的库仍可用。 可以通过将 `spark.databricks.libraryIsolation.enabled` 设置为 `false` 来禁用此功能。

此 API 与通过 [UI](../libraries/index.md) 和 [REST API](api/latest/libraries.md) 进行的群集范围的现有库安装兼容。 通过此 API 安装的库的优先级高于群集范围的库。

```python
dbutils.library.help()
```

```
install(path: String): boolean -> Install the library within the current notebook session
installPyPI(pypiPackage: String, version: String = "", repo: String = "", extras: String = ""): boolean -> Install the PyPI library within the current notebook session
list: List -> List the isolated libraries added for the current notebook session via dbutils
restartPython: void -> Restart python process for the current notebook session
```

> [!NOTE]
>
> 可以使用 `help` 获取有关每个命令的详细信息，例如 `dbutils.library.help("install")`

### <a name="examples"></a>示例

* 在笔记本中安装 PyPI 库。 `version`、`repo` 和 `extras` 是可选的。 使用 `extras` 参数可指定[额外功能](https://setuptools.readthedocs.io/en/latest/setuptools.html#declaring-extras-optional-features-with-their-own-dependencies)（额外要求）。

  ```python
  dbutils.library.installPyPI("pypipackage", version="version", repo="repo", extras="extras")
  dbutils.library.restartPython()  # Removes Python state, but some libraries might not work without calling this function
  ```

  > [!IMPORTANT]
  >
  > `version` 和 `extras` 键不能是 PyPI 包字符串的一部分。 例如，`dbutils.library.installPyPI("azureml-sdk[databricks]==1.0.8")` 无效。 请使用 `version` 和 `extras` 参数指定版本和额外信息，如下所示：
  >
  > ```python
  > dbutils.library.installPyPI("azureml-sdk", version="1.0.8", extras="databricks")
  > dbutils.library.restartPython()  # Removes Python state, but some libraries might not work without calling this function
  > ```

* 在一个笔记本中指定库要求，然后通过 `%run` 在另一个笔记本中安装它们。
  * 定义要在名为 `InstallDependencies` 的笔记本中安装的库。

    ```python
    dbutils.library.installPyPI("torch")
    dbutils.library.installPyPI("scikit-learn", version="1.19.1")
    dbutils.library.installPyPI("azureml-sdk", extras="databricks")
    dbutils.library.restartPython()  # Removes Python state, but some libraries might not work without calling this function
    ```

  * 将它们安装在需要这些依赖项的笔记本中。

    ```bash
    %run /path/to/InstallDependencies    # Install the dependencies in first cell
    ```

    ```python
    import torch
    from sklearn.linear_model import LinearRegression
    import azureml
    # do the actual work
    ```

* 列出笔记本中已安装的库。

  ```python
  dbutils.library.list()
  ```

* 在维护环境时重置 Python 笔记本状态。 此 API 仅在 Python 笔记本中可用。 这可用于：
  * 重新加载 Azure Databricks 预安装的其他版本的库。 例如：

    ```python
    dbutils.library.installPyPI("numpy", version="1.15.4")
    dbutils.library.restartPython()
    ```

    ```python
    # Make sure you start using the library in another cell.
    import numpy
    ```

  * 安装需要在进程启动时加载的库，例如 tensorflow。 例如：

    ```python
    dbutils.library.installPyPI("tensorflow")
    dbutils.library.restartPython()
    ```

    ```python
    # Use the library in another cell.
    import tensorflow
    ```

* 在笔记本中安装 `.egg` 或 `.whl` 库。

  > [!IMPORTANT]
  >
  > 建议将所有库安装命令放入笔记本的第一个单元，在该单元的末尾调用 `restartPython`。 在运行 `restartPython` 后会重置 Python 笔记本状态；笔记本会丢失所有状态，包括但不限于本地变量、导入的库和其他临时状态。 因此，建议在第一个笔记本单元中安装库并重置笔记本状态。

  接受的库源是 `dbfs`、`abfss`、`adl` 和 `wasbs`。

  ```python
  dbutils.library.install("abfss:/path/to/your/library.egg")
  dbutils.library.restartPython()  # Removes Python state, but some libraries might not work without calling this function
  ```

  ```python
  dbutils.library.install("abfss:/path/to/your/library.whl")
  dbutils.library.restartPython()  # Removes Python state, but some libraries might not work without calling this function
  ```

#### <a name="dbutils-notebook"></a>DBUtils 笔记本

[获取笔记本](../_static/notebooks/dbutils.html)

## <a name="databricks-utilities-api-library"></a><a id="databricks-utilities-api-library"> </a><a id="dbutils-api"> </a>Databricks 实用工具 API 库

> [!NOTE]
>
> 如果要部署到运行 Databricks Runtime 7.0 或更高版本的群集，则无法使用此库来运行部署前测试，因为没有支持 Scala 2.12 的库版本。

为了加快应用程序开发，可以在将应用程序部署为生产作业之前对其进行编译、构建和测试。 为了让你能够针对 Databricks 实用工具进行编译，Databricks 提供了 `dbutils-api` 库。 可以[下载 dbutils-api 库](https://dl.bintray.com/databricks/maven/dbutils-api.jar)，也可以通过将依赖项添加到生成文件来包含库：

* SBT

  ```scala
  libraryDependencies += "com.databricks" % "dbutils-api_2.11" % "0.0.4"
  ```

* Maven

  ```xml
  <dependency>
      <groupId>com.databricks</groupId>
      <artifactId>dbutils-api_2.11</artifactId>
      <version>0.0.4</version>
  </dependency>
  ```

* Gradle

  ```bash
  compile 'com.databricks:dbutils-api_2.11:0.0.4'
  ```

针对此库生成应用程序后，即可部署该应用程序。

> [!IMPORTANT]
>
> `dbutils-api` 库允许你在本地编译一个使用 `dbutils` 的应用程序，但不允许你运行它。 若要运行该应用程序，必须将其部署到 Azure Databricks 中。

### <a name="express-the-artifacts-scala-version-with-"></a>使用 `%%` 表示生成工件的 Scala 版本

如果将生成工件版本表示为 `groupID %% artifactID % revision` 而不是 `groupID % artifactID % revision`（区别在于 `groupID` 后面的双 `%%`），则 SBT 会将项目的 Scala 版本添加到生成工件名称中。

**示例**

假设你的生成的 `scalaVersion` 为 `2.9.1`。 可以使用 `%` 编写生成工件版本，如下所示：

```scala
val appDependencies = Seq(
  "org.scala-tools" % "scala-stm_2.9.1" % "0.3"
)
```

下面这个使用 `%%` 的语句是等效的：

```scala
val appDependencies = Seq(
  "org.scala-tools" %% "scala-stm" % "0.3"
)
```

### <a name="example-projects"></a>示例项目

下面是一个[示例存档](https://dl.bintray.com/databricks/maven/dbutils-maven.zip)，其中包含最小的示例项目，用于演示如何使用适用于 3 个常用生成工具的 `dbutils-api` 库进行编译：

* sbt：`sbt package`
* Maven：`mvn install`
* Gradle：`gradle build`

这些命令在以下位置创建输出 JAR：

* sbt：`target/scala-2.11/dbutils-api-example_2.11-0.0.1-SNAPSHOT.jar`
* Maven：`target/dbutils-api-example-0.0.1-SNAPSHOT.jar`
* Gradle：`build/examples/dbutils-api-example-0.0.1-SNAPSHOT.jar`

可以将此 JAR 作为库附加到群集，重启群集，然后运行以下语句：

```scala
example.Test()
```

此语句创建一个文本输入小组件，其标签为 Hello:，初始值为 World。

可以通过相同的方式使用所有其他 `dbutils` API。

若要测试某个在 Databricks 外部使用 `dbutils` 对象的应用程序，可以通过调用以下语句来模拟 `dbutils` 对象：

```scala
com.databricks.dbutils_v1.DBUtilsHolder.dbutils0.set(
  new com.databricks.dbutils_v1.DBUtilsV1{
    ...
  }
)
```

请使用你自己的 `DBUtilsV1` 实例进行替换。在该实例中，你可以随意实现接口方法，例如，为 `dbutils.fs` 提供本地文件系统原型。