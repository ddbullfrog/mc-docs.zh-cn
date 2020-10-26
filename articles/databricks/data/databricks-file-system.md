---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/16/2020
title: Databricks 文件系统 (DBFS) - Azure Databricks
description: 了解 Databricks 文件系统 (DBFS)。
ms.openlocfilehash: 5ef571420862660378469b76afda066fb93651f2
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121859"
---
# <a name="databricks-file-system-dbfs"></a>Databricks 文件系统 (DBFS)

Databricks 文件系统 (DBFS) 是一个装载到 Azure Databricks 工作区的分布式文件系统，可以在 Azure Databricks 群集上使用。 DBFS 是基于可缩放对象存储的抽象，具有以下优势：

* 允许你[装载](#mount-storage)存储对象，因此无需凭据即可无缝访问数据。
* 允许你使用目录和文件语义（而不是存储 URL）与对象存储进行交互。
* 将文件保存到对象存储，因此在终止群集后不会丢失数据。

## <a name="dbfs-root"></a>DBFS 根

DBFS 中的默认存储位置称为 _DBFS 根_。 以下 DBFS 根位置中存储了几种类型的数据：

* `/FileStore`：导入的数据文件、生成的绘图以及上传的库。 请参阅 [FileStore](filestore.md#filestore)。
* `/databricks-datasets`：示例[公共数据集](databricks-datasets.md#databricks-datasets)。
* `/databricks-results`：通过下载查询的[完整结果](../notebooks/notebooks-use.md#download-full-results)生成的文件。
* `/databricks/init`：全局和群集命名的（已弃用）[init 脚本](../clusters/init-scripts.md)。
* `/user/hive/warehouse`：非外部 Hive 表的数据和元数据。

在新的工作区中，DBFS 根具有以下默认文件夹：

> [!div class="mx-imgBorder"]
> ![DBFS 根默认文件](../_static/images/getting-started/dbfs-root.png)

DBFS 根还包含不可见且无法直接访问的数据，包括装入点元数据和凭据以及某些类型的日志。

> [!IMPORTANT]
>
> 已写入[装入点路径](#mount-storage) (`/mnt`) 的数据存储在 DBFS 根的外部。 尽管 DBFS 根是可写的，仍建议你将数据存储在装载的对象存储中，而不是存储在 DBFS 根中。

## <a name="mount-object-storage-to-dbfs"></a><a id="mount-object-storage-to-dbfs"> </a><a id="mount-storage"> </a>将对象存储装载到 DBFS

通过将对象存储装载到 DBFS，可访问对象存储中的对象，就像它们在本地文件系统中一样。

> [!IMPORTANT]
>
> * 所有用户都对装载到 DBFS 的对象存储中的对象具有读写访问权限。
> * 不支持嵌套装载。 例如，不支持以下结构：
>   * `storage1` 装载为 `/mnt/storage1`
>   * `storage2` 装载为 `/mnt/storage1/storage2`
>
>   建议为每个存储对象创建单独的装载条目：
>
>   * `storage1` 装载为 `/mnt/storage1`
>   * `storage2` 装载为 `/mnt/storage2`

要了解如何装载和卸载 Azure Blob 存储容器和 Azure Data Lake Storage 帐户，请参阅[将 Azure Blob 存储容器装载到 DBFS](data-sources/azure/azure-storage.md#mount-azure-blob-storage)、[使用凭据传递将 Azure Data Lake Storage 装载到 DBFS](../security/credential-passthrough/adls-passthrough.md#aad-passthrough-dbfs) 以及[使用服务主体和 OAuth 2.0 装载 Azure Data Lake Storage Gen2 帐户](data-sources/azure/azure-datalake-gen2.md#mount-azure-data-lake-gen2)。

## <a name="access-dbfs"></a>访问 DBFS

可使用[文件上传接口](#user-interface)将数据上传到 DBFS，并使用 [DBFS CLI](../dev-tools/cli/dbfs-cli.md)、[DBFS API](../dev-tools/api/latest/dbfs.md)、[Databricks 文件系统实用工具 (dbutils.fs)](../dev-tools/databricks-utils.md#dbutils-fs)、[Spark API](#dbfs-spark) 和[本地文件 API](#fuse) 来上传和访问 DBFS 对象。 在 Spark 群集中，使用 Databricks 文件系统实用工具、Spark API 或本地文件 API 访问 DBFS 对象。 在本地计算机上，使用 Databricks CLI 或 DBFS API 访问 DBFS 对象。

### <a name="in-this-section"></a>本节内容：

* [文件上传接口](#file-upload-interface)
* [Databricks CLI](#databricks-cli)
* [`dbutils`](#dbutils)
* [DBFS API](#dbfs-api)
* [Spark API](#spark-apis)
* [本地文件 API](#local-file-apis)
* [用于深度学习的本地文件 API](#local-file-apis-for-deep-learning)

### <a name="file-upload-interface"></a><a id="file-upload-interface"> </a><a id="user-interface"> </a>文件上传接口

如果本地计算机上有要使用 Azure Databricks 进行分析的小型数据文件，可使用文件上传接口将其轻松导入 [Databricks 文件系统 (DBFS)]()。

> [!NOTE]
>
> 管理员用户可禁用文件上传接口。 请查看[管理数据上传](../administration-guide/workspace/dbfs-ui-upload.md)。

如果要使用 UI 创建[表](tables.md)，请参阅[使用 UI 创建表](tables.md#create-a-table-using-the-ui)。

如果要上传在笔记本中使用的数据，请执行以下步骤。

1. 创建新笔记本或打开现有笔记本，然后单击“文件”>“上传数据”

   > [!div class="mx-imgBorder"]
   > ![上传数据](../_static/images/notebooks/upload-data-menu.png)

2. 选择 DBFS 中的目标目录来存储已上传的文件。 目标目录默认为 `/shared_uploads/<your-email-address>/`。

   具有工作区访问权限的用户都可访问已上传的文件。

3. 将文件拖放到放置目标上，或者单击“浏览”以查找本地文件系统中的文件。

   > [!div class="mx-imgBorder"]
   > ![选择文件和目标](../_static/images/notebooks/upload-data-upload-step_2.png)

4. 完成文件上传后，单击“下一步”。

   如果已上传 CSV、TSV 或 JSON 文件，Azure Databricks 会生成代码，显示如何将数据加载到数据帧。

   > [!div class="mx-imgBorder"]
   > ![查看文件和示例代码](../_static/images/notebooks/upload-data-view-files-sample-code_2.png)

   若要将文本保存到剪贴板，请单击“复制”。

5. 单击“完成”，返回到笔记本。

### <a name="databricks-cli"></a>Databricks CLI

DBFS 命令行接口 (CLI) 使用 [DBFS API](../dev-tools/api/latest/dbfs.md) 向 DBFS 公开一个易用型命令行接口。 通过此客户端，你可使用类似于 Unix 命令行中所用的命令与 DBFS 进行交互。 例如： 。

```bash
# List files in DBFS
dbfs ls
# Put local file ./apple.txt to dbfs:/apple.txt
dbfs cp ./apple.txt dbfs:/apple.txt
# Get dbfs:/apple.txt and save to local file ./apple.txt
dbfs cp dbfs:/apple.txt ./apple.txt
# Recursively put local dir ./banana to dbfs:/banana
dbfs cp -r ./banana dbfs:/banana
```

若要详细了解 DBFS 命令行接口，请参阅 [Databricks CLI](../dev-tools/cli/index.md)。

### <a name="dbutils"></a><a id="dbfs-dbutils"> </a><a id="dbutils"> </a>`dbutils`

[dbutils.fs](../dev-tools/databricks-utils.md#dbutils-fs) 提供与文件系统类似的命令来访问 DBFS 中的文件。
本部分提供几个示例，说明如何使用 `dbutils.fs` 命令在 DBFS 中写入和读取文件。

> [!TIP]
>
> 若要访问 DBFS 的帮助菜单，请使用 `dbutils.fs.help()` 命令。

* 在 DBFS 根中写入和读取文件，就像它是本地文件系统一样。

  ```python
  dbutils.fs.mkdirs("/foobar/")
  ```

  ```python
  dbutils.fs.put("/foobar/baz.txt", "Hello, World!")
  ```

  ```python
  dbutils.fs.head("/foobar/baz.txt")
  ```

  ```python
  dbutils.fs.rm("/foobar/baz.txt")
  ```

* 使用 `dbfs:/` 访问 DBFS 路径。

  ```python
  display(dbutils.fs.ls("dbfs:/foobar"))
  ```

* 笔记本支持使用速记命令（`%fs` [魔术命令](../notebooks/notebooks-use.md#language-magic)）来访问 `dbutils` 文件系统模块。 可通过 `%fs` 魔术命令使用大多数 `dbutils.fs` 命令。

  ```bash
  # List the DBFS root

  %fs ls

  # Recursively remove the files under foobar

  %fs rm -r foobar

  # Overwrite the file "/mnt/my-file" with the string "Hello world!"

  %fs put -f "/mnt/my-file" "Hello world!"
  ```

### <a name="dbfs-api"></a>DBFS API

请参阅 [DBFS API](../dev-tools/api/latest/dbfs.md) 和[将大文件上传到 DBFS](../dev-tools/api/latest/examples.md#dbfs-large-files)。

### <a name="spark-apis"></a><a id="dbfs-spark"> </a><a id="spark-apis"> </a>Spark API

使用 Spark API 时，将使用 `"/mnt/training/file.csv"` 或 `"dbfs:/mnt/training/file.csv"` 引用文件。  以下示例将文件 `foo.text` 写入 DBFS `/tmp` 目录。

```python
df.write.text("/tmp/foo.txt")
```

### <a name="local-file-apis"></a><a id="fuse"> </a><a id="local-file-apis"> </a>本地文件 API

可使用本地文件 API 来读取和写入 DBFS 路径。 Azure Databricks 使用 FUSE 装载 `/dbfs` 配置每个群集节点，使群集节点上运行的进程能够使用本地文件 API 来读取和写入基础分布式存储层。 使用本地文件 API 时，必须在 `/dbfs` 下提供路径。  例如： 。

#### <a name="python"></a>Python

```python
#write a file to DBFS using Python I/O APIs
with open("/dbfs/tmp/test_dbfs.txt", 'w') as f:
  f.write("Apache Spark is awesome!\n")
  f.write("End of example!")

# read the file
with open("/dbfs/tmp/test_dbfs.txt", "r") as f_read:
  for line in f_read:
    print line
```

#### <a name="scala"></a>Scala

```scala
import scala.io.Source

val filename = "/dbfs/tmp/test_dbfs.txt"
for (line <- Source.fromFile(filename).getLines()) {
  println(line)
}
```

#### <a name="local-file-api-limitations"></a><a id="local-file-api-limitations"> </a><a id="local-limitations"> </a>本地文件 API 限制

下面列举了适用于 Databricks Runtime 各版本的本地文件 API 使用限制。

* **全部**：不支持凭据传递。

* **6.0**
  * 不支持随机写入。   对于需要随机写入的工作负载，请先在本地磁盘上执行 I/O，然后将结果复制到 `/dbfs`。 例如： 。

    ```python
    # python
    import xlsxwriter
    from shutil import copyfile

    workbook = xlsxwriter.Workbook('/local_disk0/tmp/excel.xlsx')
    worksheet = workbook.add_worksheet()
    worksheet.write(0, 0, "Key")
    worksheet.write(0, 1, "Value")
    workbook.close()

    copyfile('/local_disk0/tmp/excel.xlsx', '/dbfs/tmp/excel.xlsx')
    ```

  * 不支持稀疏文件。 若要复制稀疏文件，请使用 `cp --sparse=never`：

    ```bash
    $ cp sparse.file /dbfs/sparse.file
    error writing '/dbfs/sparse.file': Operation not supported
    $ cp --sparse=never sparse.file /dbfs/sparse.file
    ```

* **5.5**
  * 仅支持小于 2 GB 的文件。 如果使用本地文件 I/O API 读取或写入大于 2 GB 的文件，则可能会导致文件损坏。 相反，请使用 [DBFS CLI](../dev-tools/cli/dbfs-cli.md)、[dbutils.fs](../dev-tools/databricks-utils.md#dbutils-fs) 或 Spark API 来访问大于 2 GB 的文件，或使用[用于深度学习的本地文件 API](#mlfuse) 中所述的 `/dbfs/ml` 文件夹。
  * 如果使用本地文件 I/O API 来写入文件，然后立即尝试使用 [DBFS CLI](../dev-tools/cli/dbfs-cli.md)、[dbutils.fs](#dbfs-dbutils) 或 Spark API 来访问它，则可能会遇到 `FileNotFoundException`、文件大小为 0 或文件内容陈旧的情况。 这是预料之中的，因为 OS 默认情况下会缓存写入。 若要强制将这些写入刷新到持久存储（在我们的示例中为 DBFS）中，请使用标准的 Unix 系统调用[同步](https://en.wikipedia.org/wiki/Sync_(Unix))。例如：

    ```scala
    // scala
    import scala.sys.process._

    // Write a file using the local file API (over the FUSE mount).
    dbutils.fs.put("file:/dbfs/tmp/test", "test-contents")

    // Flush to persistent storage.
    "sync /dbfs/tmp/test" !

    // Read the file using "dbfs:/" instead of the FUSE mount.
    dbutils.fs.head("dbfs:/tmp/test")
    ```

### <a name="local-file-apis-for-deep-learning"></a><a id="local-file-apis-for-deep-learning"> </a><a id="mlfuse"> </a>用于深度学习的本地文件 API

[分布式深度学习应用程序](../applications/machine-learning/load-data/ddl-data.md)需要访问 DBFS 来对数据进行加载、检查点和日志记录处理。对于这些应用，Databricks Runtime 6.0 及更高版本提供了一个已针对深度学习工作负载优化的高性能 `/dbfs` 装载。

在 Databricks Runtime 5.5 LTS 中，仅优化 `/dbfs/ml`。 在此版本中，Databricks 建议将数据保存在 `/dbfs/ml` 下，该数据映射到 `dbfs:/ml`。