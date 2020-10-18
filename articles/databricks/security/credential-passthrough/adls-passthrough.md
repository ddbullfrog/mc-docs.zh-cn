---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/16/2020
title: 使用 Azure Active Directory 凭据直通身份验证保护对 Azure Data Lake Storage 的访问 - Azure Databricks
description: 了解如何使用直通身份验证通过 Azure Databricks 从 Azure Data Lake Storage 读取数据以及将数据写入 Azure Data Lake Storage。
ms.openlocfilehash: 766fe315803eb781a82431a41eb9ca182d0ce696
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937813"
---
# <a name="secure-access-to-azure-data-lake-storage-using-azure-active-directory-credential-passthrough"></a>使用 Azure Active Directory 凭据直通身份验证保护对 Azure Data Lake Storage 的访问

可以使用登录 Azure Databricks 时所用的相同 Azure Active Directory (Azure AD) 标识自动从 Azure Databricks 群集向 [Azure Data Lake Storage Gen1](../../data/data-sources/azure/azure-datalake.md#adls-gen1) 和 [Azure Data Lake Storage Gen2](../../data/data-sources/azure/azure-datalake-gen2.md#adls-gen2) 进行身份验证。 为群集启用 Azure Data Lake Storage 凭据直通身份验证时，在该群集上运行的命令可以在 Azure Data Lake Storage 中读取和写入数据，无需配置用于访问存储的服务主体凭据。

## <a name="requirements"></a>要求

* Azure Data Lake Storage Gen1 或 Gen2 存储帐户。 Azure Data Lake Storage Gen2 存储帐户必须使用分层命名空间才能与 Azure Data Lake Storage 凭据直通身份验证配合使用。 请参阅[创建 Azure Data Lake Storage Gen2 帐户并初始化文件系统](../../data/data-sources/azure/azure-datalake-gen2.md#create-adls-account)。
* [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)。
* 为 Azure Databricks 用户正确配置了权限，使其能够在 Azure Data Lake Storage 中根据需要读取和写入数据。 这是 Azure Databricks 管理员的任务。

> [!IMPORTANT]
>
> 如果你位于防火墙后面，但该防火墙尚未配置为允许将流量发往 Azure Active Directory，那么你不能使用 Azure Active Directory 凭据向 Azure Data Lake Storage 进行身份验证。 Azure 防火墙默认阻止 Active Directory 访问。 若要允许访问，请配置 AzureActiveDirectory 服务标记。 可以在 Azure IP 范围和服务标记 JSON 文件中的 AzureActiveDirectory 标记下找到网络虚拟设备的等效信息。 有关详细信息，请参阅 [Azure 防火墙服务标记](/firewall/service-tags)和[公有云的 Azure IP 地址](https://www.microsoft.com/download/confirmation.aspx?id=56519)。

### <a name="cluster-requirements"></a>群集要求

* Databricks Runtime 6.0 或更高版本（用于在标准群集上提供 R 支持）。
* 不能已为群集设置了 Azure Data Lake Storage 凭据（例如，通过提供服务主体凭据进行设置）。
* 启用了凭据直通身份验证的群集不支持作业，也不支持[数据对象特权](../access-control/table-acls/object-privileges.md)。

## <a name="logging-recommendations"></a>日志记录建议

由于你的标识会传递到 Azure Data Lake Storage，因此你的标识可以出现在 Azure 存储诊断日志中。 因此，可以将 ADLS 请求绑定到 Azure Databricks 群集中的单个用户。 若要开始接收这些日志，请在存储帐户上启用诊断日志记录功能。

* Azure Data Lake Storage Gen1：按照[对 Data Lake Storage Gen1 帐户启用诊断日志记录](/data-lake-store/data-lake-store-diagnostic-logs#enable-diagnostic-logging-for-your-data-lake-storage-gen1-account)中的说明操作。
* Azure Data Lake Storage Gen2：使用 PowerShell 执行 `Set-AzStorageServiceLoggingProperty` 命令进行配置。 指定 2.0 作为版本，因为日志条目格式 2.0 包含请求中的用户主体名称。

## <a name="enable-azure-data-lake-storage-credential-passthrough-for-a-high-concurrency-cluster"></a>为高并发群集启用 Azure Data Lake Storage 凭据直通身份验证

高并发性群集可由多个用户共享。 它们仅支持 Python、SQL 和 R。

1. [创建群集](../../clusters/create.md#cluster-create)时，请将“群集模式”设置为[高并发性](../../clusters/configure.md#high-concurrency)。
2. 在“高级选项”下，选择“启用凭据直通身份验证，但只允许 Python 和 SQL 命令”。

> [!div class="mx-imgBorder"]
> ![启用凭据直通身份验证](../../_static/images/clusters/adls-credential-passthrough.png)

## <a name="enable-azure-data-lake-storage-credential-passthrough-for-a-standard-cluster"></a><a id="enable-azure-data-lake-storage-credential-passthrough-for-a-standard-cluster"> </a><a id="single-user"> </a>为标准群集启用 Azure Data Lake Storage 凭据直通身份验证

对于启用了凭据直通身份验证的标准群集，只能进行单用户访问。 标准群集支持 Python、SQL 和 Scala。 在 Databricks Runtime 6.0 及更高版本上，它们也支持 SparkR。

你必须在群集创建时分配一个用户，但该群集随时可由具有“可管理”权限的用户编辑，以替换原始用户。

> [!IMPORTANT]
>
> 分配给群集的用户必须至少有群集的“可附加到”权限，才能在群集上运行命令。 管理员和群集创建者有“可管理”权限，但不能在群集上运行命令，除非他们是指定的群集用户。

1. [创建群集](../../clusters/create.md#cluster-create)时，请将“群集模式”设置为[标准](../../clusters/configure.md#standard)。
2. 在“高级选项”下，选择“为用户级访问启用凭据直通身份验证”，然后从“单用户访问”下拉列表中选择用户名。

> [!div class="mx-imgBorder"]
> ![启用凭据直通身份验证](../../_static/images/clusters/credential-passthrough-single.png)

## <a name="read-and-write-azure-data-lake-storage-using-credential-passthrough"></a>使用凭据直通身份验证读取和写入 Azure Data Lake Storage

Azure Data Lake Storage 凭据直通身份验证仅支持 Azure Data Lake Storage Gen1 和 Gen2。 在 Azure Data Lake Storage Gen1 中使用 `adl://` 路径直接访问数据，在 Azure Data Lake Storage Gen2 中使用 `abfss://` 路径直接访问数据。 例如：

### <a name="azure-data-lake-storage-gen1"></a>Azure Data Lake Storage Gen1

```python
spark.read.csv("adl://<myadlsfolder>.azuredatalakestore.net/MyData.csv").collect()
```

### <a name="azure-data-lake-storage-gen2"></a>Azure Data Lake Storage Gen2

```python
spark.read.csv("abfss://<my-file-system-name>@<my-storage-account-name>.dfs.core.chinacloudapi.cn/MyData.csv").collect()
```

## <a name="mount-azure-data-lake-storage-to-dbfs-using-credential-passthrough"></a><a id="aad-passthrough-dbfs"> </a><a id="mount-azure-data-lake-storage-to-dbfs-using-credential-passthrough"> </a>使用凭据直通身份验证将 Azure Data Lake Storage 装载到 DBFS

可以将 Azure Data Lake Storage 帐户或其中的文件夹装载到 [Databricks 文件系统 (DBFS)](../../data/databricks-file-system.md)。 此装载是指向一个数据湖存储的指针，因此数据永远不会在本地同步。

使用启用了 Azure data Lake Storage 凭据直通身份验证的群集装载数据时，对装入点的任何读取或写入操作都将使用 Azure AD 凭据。 此装入点对其他用户可见，但只有以下用户具有读取和写入访问权限：

* 有权访问基础 Azure Data Lake Storage 存储帐户的用户
* 正在使用已启用 Azure Data Lake Storage 凭据直通身份验证的群集的用户

若要装载 Azure Data Lake Storage 帐户或其中的文件夹，请使用[装载 Azure Data Lake Storage Gen1 资源或文件夹](../../data/data-sources/azure/azure-datalake.md#mount-adls)或[装载 Azure Data Lake Storage Gen2 文件系统](../../data/data-sources/azure/azure-datalake-gen2.md#mount-adls-gen2)中所述的 Python 命令，并且将 `configs` 替换为以下内容：

### <a name="azure-data-lake-storage-gen1"></a>Azure Data Lake Storage Gen1

```python
configs = {
  "fs.adl.oauth2.access.token.provider.type": "CustomAccessTokenProvider",
  "fs.adl.oauth2.access.token.custom.provider": spark.conf.get("spark.databricks.passthrough.adls.tokenProviderClassName")
}
```

> [!NOTE]
>
> 从 Databricks Runtime 6.0 开始，我们已弃用 Azure Data Lake Storage 配置键的 `dfs.adls.` 前缀，改用新的 `fs.adl.` 前缀。 但保留了后向兼容性，这意味着你仍然可以使用旧的前缀。 不过，使用旧前缀时有两个注意事项。 第一个注意事项是，即使使用旧前缀的键会正确传播，使用带新前缀的键调用 `spark.conf.get` 也会失败，除非进行了显式设置。 第二个注意事项是，任何引用 Azure Data Lake Storage 配置键的错误消息都将始终使用新前缀。 对于低于 6.0 的 Databricks Runtime 版本，必须始终使用旧前缀。

### <a name="azure-data-lake-storage-gen2"></a>Azure Data Lake Storage Gen2

```python
configs = {
  "fs.azure.account.auth.type": "CustomAccessToken",
  "fs.azure.account.custom.token.provider.class":   spark.conf.get("spark.databricks.passthrough.adls.gen2.tokenProviderClassName")
}
```

> [!WARNING]
>
> 不要提供存储帐户访问密钥或服务主体凭据来向装入点进行身份验证。 那样会使得其他用户可以使用这些凭据访问文件系统。 Azure Data Lake Storage 凭据直通身份验证的目的是让你不必使用这些凭据，以及确保只有有权访问基础 Azure Data Lake Storage 帐户的用户才能访问文件系统。

## <a name="security"></a>安全

与其他用户共享 Azure Data Lake Storage 凭据直通身份验证群集是安全的。 你和其他用户彼此独立，无法读取或使用彼此的凭据。

## <a name="supported-features"></a>支持的功能

| 功能                                                                          | 最低 Databricks Runtime 版本 | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|----------------------------------------------------------------------------------|------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Python 和 SQL                                                                   | 5.1                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Azure Data Lake Storage Gen1                                                     | 5.1                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `%run`                                                                           | 5.1                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| DBFS                                                                             | 5.3                                | 仅当 DBFS 路径解析为 Azure Data Lake Storage Gen1 或 Gen2 中的位置时，才会传递凭据。 对于会解析为其他存储系统的 DBFS 路径，请使用另一方法来指定凭据。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Azure Data Lake Storage Gen2                                                     | 5.3                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| [增量缓存](../../delta/optimizations/delta-cache.md)                        | 5.4                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| PySpark ML API                                                                   | 5.4                                | **不支持以下 ML 类：**<br><br>* `org/apache/spark/ml/classification/RandomForestClassifier`<br>* `org/apache/spark/ml/clustering/BisectingKMeans`<br>* `org/apache/spark/ml/clustering/GaussianMixture`<br>* `org/spark/ml/clustering/KMeans`<br>* `org/spark/ml/clustering/LDA`<br>* `org/spark/ml/evaluation/ClusteringEvaluator`<br>* `org/spark/ml/feature/HashingTF`<br>* `org/spark/ml/feature/OneHotEncoder`<br>* `org/spark/ml/feature/StopWordsRemover`<br>* `org/spark/ml/feature/VectorIndexer`<br>* `org/spark/ml/feature/VectorSizeHint`<br>* `org/spark/ml/regression/IsotonicRegression`<br>* `org/spark/ml/regression/RandomForestRegressor`<br>* `org/spark/ml/util/DatasetUtils` |
| 广播变量                                                              | 5.5                                | 在 PySpark 中，可以构造的 Python UDF 的大小存在限制，因为大型 UDF 是作为广播变量发送的。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| [作用域为笔记本的库](../../dev-tools/databricks-utils.md#dbutils-library) | 5.5                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Scala                                                                            | 5.5                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Spark R                                                                          | 6.0                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| [笔记本工作流](../../notebooks/notebook-workflows.md)                      | 6.1                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| PySpark ML API                                                                   | 6.1                                | 所有 PySpark ML 类都受支持。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| [Ganglia UI](../../clusters/clusters-manage.md#ganglia-metrics)                  | 6.1                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

## <a name="limitations"></a>限制

Azure Data Lake Storage 凭据直通身份验证不支持以下功能：

* `%fs`（请改用等效的 [dbutils.fs](../../dev-tools/databricks-utils.md#dbutils-fs) 命令）。
* [作业](../../jobs.md)。
* [REST API](../../dev-tools/api/index.md)。
* [表访问控制](../access-control/table-acls/object-privileges.md)。 Azure Data Lake Storage 凭据直通身份验证所授予的功能可用于绕过表 ACL 的细化权限，而表 ACL 的额外限制会限制你通过 Azure Data Lake Storage 凭据直通身份验证获得的某些能力。 具体而言：
  * 如果你的 Azure AD 权限允许你访问特定表所基于的数据文件，那么你可以通过 RDD API 获取对该表的完全权限（无论通过表 ACL 对其施加的限制如何）。
  * 只有在使用数据帧 API 时，才会受到表 ACL 权限的约束。 如果你尝试直接通过数据帧 API 读取文件，则即使你可以直接通过 RDD API 读取那些文件，也会看到警告，指出你在任何文件上都没有 `SELECT` 权限。
  * 你将无法从 Azure Data Lake Storage 以外的文件系统所支持的表进行读取，即使你具有读取表的表 ACL 权限。
* SparkContext (`sc`) 和 SparkSession (`spark`) 对象上的以下方法：
  * 已弃用的方法。
  * 允许非管理员用户调用 Scala 代码的方法，例如 `addFile()` 和 `addJar()`。
  * 访问 Azure Data Lake Storage Gen1 或 Gen2 以外的文件系统的任何方法（若要访问启用了 Azure Data Lake Storage 凭据直通身份验证的群集上的其他文件系统，请使用另一方法来指定凭据，并参阅[故障排除](#aad-passthrough-troubleshoot)下关于受信任文件系统的部分）。
  * 旧的 Hadoop API（`hadoopFile()` 和 `hadoopRDD()`）。
  * 流式处理 API，因为直通凭据会在流仍在运行时过期。
* [FUSE 装载](../../data/databricks-file-system.md#fuse) (/dbfs)。
* Azure 数据工厂。
* [高并发性](../../clusters/configure.md#high-concurrency)群集上的 [Databricks Connect](../../dev-tools/databricks-connect.md)。
* 高并发性群集上的 [MLflow](../../applications/mlflow/index.md)。
* 高并发性群集上的 [azureml-sdk[databricks]](https://pypi.org/project/azureml-sdk/) Python 包。
* 不能使用 Azure Active Directory 令牌生存期策略来延长 Azure Active Directory 直通令牌的生存期。 因此，如果向群集发送耗时超过一小时的命令，并在 1 小时标记期过后访问 Azure Data Lake Storage 资源，该命令会失败。

## <a name="example-notebooks"></a>示例笔记本

以下笔记本演示 Azure Data Lake Storage Gen1 和 Gen2 的 Azure Data Lake Storage 凭据直通身份验证。

### <a name="azure-data-lake-storage-gen1-passthrough-notebook"></a>Azure Data Lake Storage Gen1 直通笔记本

[获取笔记本](../../_static/notebooks/adls-passthrough-gen1.html)

### <a name="azure-data-lake-storage-gen2-passthrough-notebook"></a>Azure Data Lake Storage Gen2 直通笔记本

[获取笔记本](../../_static/notebooks/adls-passthrough-gen2.html)

## <a name="troubleshooting"></a><a id="aad-passthrough-troubleshoot"> </a><a id="troubleshooting"> </a>故障排除

**py4j.security.Py4JSecurityException: … 未加入允许列表**

如果访问的方法未被 Azure Databricks 明确标记为对 Azure Data Lake Storage 凭据直通身份验证群集安全，则将引发此异常。 在大多数情况下，这意味着该方法可能会允许 Azure Data Lake Storage 凭据直通身份验证群集上的用户访问其他用户的凭据。

**org.apache.spark.api.python.PythonSecurityException:路径 … 使用不受信任的文件系统**

如果你尝试访问了一个文件系统，而 Azure Data Lake Storage 凭据直通身份验证群集不知道该文件系统是否安全，则会引发此异常。 使用不受信任的文件系统可能会使 Azure Data Lake Storage 凭据直通身份验证群集上的用户能够访问其他用户的凭据，因此我们禁用了我们无法确信其会被用户安全使用的所有文件系统。

若要在 Azure Data Lake Storage 凭据直通身份验证群集上配置一组受信任的文件系统，请将该群集上的 Spark conf 键 `spark.databricks.pyspark.trustedFilesystems` 设置为以逗号分隔的类名称列表，这些名称是 `org.apache.hadoop.fs.FileSystem` 的受信任实现。

<!--comment-out https://databricks.atlassian.net/browse/SC-20253
To get the list of trusted file systems on a cluster enabled for |Passthrough|, run this command:

```python

  spark.conf.get("spark.databricks.pyspark.trustedFilesystems")

.. note::

  - In <DBR> 5.1, Azure Blob storage is *not* trusted by default. To trust Azure Blob storage in <DBR> 5.1, you must manually configure it to be trusted.

  - In <DBR> 5.2 and above, Azure Blob storage is trusted by default, and you do not need to configure it to be trusted.-->