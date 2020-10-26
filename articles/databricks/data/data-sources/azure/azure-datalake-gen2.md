---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: Azure Data Lake Storage Gen2 - Azure Databricks
description: 了解如何通过使用 Azure Databricks 来进行身份验证、读取数据以及将数据写入 Azure Data Lake Storage Gen2。
ms.openlocfilehash: 66f3253c972930c5618c108235b3d11891befec3
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121884"
---
# <a name="azure-data-lake-storage-gen2"></a><a id="adls-gen2"> </a><a id="azure-data-lake-storage-gen2"> </a>Azure Data Lake Storage Gen2

[Azure Data Lake Storage Gen2](/storage/data-lake-storage/introduction)（也称为 ADLS Gen2）是用于大数据分析的下一代 [data lake](https://databricks.com/discover/data-lakes/introduction) 解决方案。 Azure Data Lake Storage Gen2 将 Azure Data Lake Storage Gen1 功能（文件系统语义、文件级安全性和扩展）构建到 Azure Blob 存储中，并具有低成本分层存储、高可用性和灾难恢复功能。

有四种方法可以访问 Azure Data Lake Storage Gen2：

1. 传递 Azure Active Directory 凭据，也称为[凭据传递](#adls2-aad-credentials)。
2. 使用服务主体和 OAuth 2.0 将 Azure Data Lake Storage Gen2 文件系统装载到 DBFS。
3. 直接使用服务主体。
4. 直接使用 Azure Data Lake Storage Gen2 存储帐户访问密钥。

本文介绍如何通过使用 Databricks Runtime 内置 [Azure Blob File System (ABFS) 驱动程序](/storage/data-lake-storage/abfs-driver)来访问 Azure Data Lake Storage Gen2。 还涵盖可以访问 Azure Data Lake Storage Gen2 的所有方式、常见问题和已知问题。

## <a name="create-an-azure-data-lake-storage-gen2-account-and-initialize-a-filesystem"></a><a id="create-adls-account"> </a><a id="create-an-azure-data-lake-storage-gen2-account-and-initialize-a-filesystem"> </a>创建 Azure Data Lake Storage Gen2 帐户并初始化文件系统

如果要使用 Azure Data Lake Storage 凭据传递或装载 Azure Data Lake Storage Gen2 文件系统，并且尚未创建 Azure Data Lake Storage Gen2 帐户和初始化文件系统，请执行以下操作：

1. [创建 Azure Data Lake Storage Gen2 存储帐户](/storage/data-lake-storage/quickstart-create-account)并启用[分层命名空间](/storage/data-lake-storage/namespace)，这有助于改进用于分析引擎和框架所熟悉的文件系统性能、POSIX ACL 和文件系统语义。

   > [!IMPORTANT]
   > * 为 Azure Data Lake Storage Gen2 帐户启用分层命名空间时，不需要通过 Azure 门户创建任何 Blob 容器。
   > * 启用分层命名空间时，Azure Blob 存储 API 不可用。 请参阅此[已知问题说明](/storage/data-lake-storage/known-issues#blob-storage-apis)。 例如，不能使用 `wasb` 和 `wasbs` 方案来访问 `blob.core.chinacloudapi.cn` 终结点。
   > * 如果启用分层命名空间，则 Azure Blob 存储和 Azure Data Lake Storage Gen2 REST API 之间将不存在数据或操作的互操作性。

2. 必须先初始化文件系统，然后才能访问它。 如果尚未从 Azure 门户中初始化文件系统，请在笔记本的第一个单元格中输入以下内容（使用帐户值）：

   ```scala
   spark.conf.set("fs.azure.createRemoteFileSystemDuringInitialization", "true")
   dbutils.fs.ls("abfss://<file-system-name>@<storage-account-name>.dfs.core.chinacloudapi.cn/")
   spark.conf.set("fs.azure.createRemoteFileSystemDuringInitialization", "false")
   ```

   每个文件系统只需运行一次，而不需每次都运行笔记本或附加到新群集。

   还可以使用 OAuth 2 对文件系统初始化进行身份验证。

   ```scala
   spark.conf.set(
     "fs.azure.account.key.<storage-account-name>.dfs.core.chinacloudapi.cn",
     dbutils.secrets.get(scope="<scope-name>",key="<storage-account-access-key-name>"))
   ```

   其中 `<storage-account-name>` 是存储帐户的名称，`dbutils.secrets.get(scope="<scope-name>",key="<storage-account-access-key-name>")` 用于检索存储帐户访问密钥，该密钥已作为[机密](../../../security/secrets/secrets.md)存储在[机密范围](../../../security/secrets/secret-scopes.md)中）。

   > [!IMPORTANT]
   >
   > Azure Data Lake Storage Gen2 文件系统验证所有提供的配置密钥，无论它们是否将用于装载或直接访问。

## <a name="access-automatically-with-your-azure-active-directory-credentials"></a><a id="access-automatically-with-your-azure-active-directory-credentials"> </a><a id="adls2-aad-credentials"> </a><a id="gen2-service-principal"> </a>使用你的 Azure Active Directory 凭据自动访问

可以配置 Azure Databricks 群集，以便使用用于登录到 Azure Databricks 的相同 Azure Active Directory (Azure AD) 标识自动对 Azure Data Lake Storage Gen2 进行身份验证。 为群集启用 Azure Data Lake Storage 凭据传递时，在该群集上运行的命令可以在 Azure Data Lake Storage Gen2 中读取和写入数据，无需配置用于访问存储的服务主体凭据。

有关完整设置和使用说明，请参阅[使用 Azure Active Directory 凭据传递保护对 Azure Data Lake Storage 的访问](../../../security/credential-passthrough/adls-passthrough.md)。

## <a name="create-and-grant-permissions-to-service-principal"></a>创建并向服务主体授予权限

如果所选访问方法需要具有足够权限的服务主体，而你没有这样的服务主体，请按照以下步骤操作：

1. [创建可访问资源的 Azure AD 应用程序](/azure-resource-manager/resource-group-create-service-principal-portal)和服务主体。 请注意以下属性：
   * `application-id`：唯一标识应用程序的 ID。
   * `directory-id`：唯一标识 Azure AD 实例的 ID。
   * `storage-account-name`：存储帐户的名称。
   * `service-credential`：一个字符串，应用程序用来证明其身份。
2. 注册服务主体，并在 Azure Data Lake Storage Gen2 帐户上授予正确的[角色分配](/storage/common/storage-auth-aad-rbac-portal?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)，如存储 Blob 数据参与者。

## <a name="mount-an-azure-data-lake-storage-gen2-account-using-a-service-principal-and-oauth-20"></a><a id="mount-an-azure-data-lake-storage-gen2-account-using-a-service-principal-and-oauth-20"> </a><a id="mount-azure-data-lake-gen2"> </a>使用服务主体和 OAuth 2.0 装载 Azure Data Lake Storage Gen2 帐户

可以使用服务主体和 OAuth 2.0 将 Azure Data Lake Storage Gen2 帐户装载到 DBFS，并进行身份验证。 此装载是指向一个数据湖存储的指针，因此数据永远不会在本地同步。

> [!IMPORTANT]
>
> * 仅支持使用 OAuth 凭据装载 Azure Data Lake Storage Gen2。 不支持使用帐户访问密钥进行装载。
> * Azure Databricks 工作区中的所有用户都有权访问已装载的 Azure Data Lake Storage Gen2 帐户。 用于访问 Azure Data Lake Storage Gen2 帐户的服务客户端应仅授予对该 Azure Data Lake Storage Gen2 帐户的访问权限；不应授予它对 Azure 中其他资源的访问权限。
> * 通过群集创建装入点后，该群集的用户可立即访问装入点。 若要在另一个正在运行的群集中使用装入点，则必须在运行的群集上运行 `dbutils.fs.refreshMounts()`，使新创建的装入点可供使用。

### <a name="mount-azure-data-lake-storage-gen2-filesystem"></a><a id="mount-adls-gen2"> </a><a id="mount-azure-data-lake-storage-gen2-filesystem"> </a>Azure Data Lake Storage Gen2 文件系统

1. 若要装载 Azure Data Lake Storage Gen2 文件系统或其内部文件夹，请使用以下命令：

   #### <a name="python"></a>Python

   ```python
   configs = {"fs.azure.account.auth.type": "OAuth",
              "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
              "fs.azure.account.oauth2.client.id": "<application-id>",
              "fs.azure.account.oauth2.client.secret": dbutils.secrets.get(scope="<scope-name>",key="<service-credential-key-name>"),
              "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/<directory-id>/oauth2/token"}

   # Optionally, you can add <directory-name> to the source URI of your mount point.
   dbutils.fs.mount(
     source = "abfss://<file-system-name>@<storage-account-name>.dfs.core.chinacloudapi.cn/",
     mount_point = "/mnt/<mount-name>",
     extra_configs = configs)
   ```

   #### <a name="scala"></a>Scala

   ```scala
   val configs = Map(
     "fs.azure.account.auth.type" -> "OAuth",
     "fs.azure.account.oauth.provider.type" -> "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
     "fs.azure.account.oauth2.client.id" -> "<application-id>",
     "fs.azure.account.oauth2.client.secret" -> dbutils.secrets.get(scope="<scope-name>",key="<service-credential-key-name>"),
     "fs.azure.account.oauth2.client.endpoint" -> "https://login.microsoftonline.com/<directory-id>/oauth2/token")

   // Optionally, you can add <directory-name> to the source URI of your mount point.
   dbutils.fs.mount(
     source = "abfss://<file-system-name>@<storage-account-name>.dfs.core.chinacloudapi.cn/",
     mountPoint = "/mnt/<mount-name>",
     extraConfigs = configs)
   ```

   where

   * `<mount-name>` 是 DBFS 路径，用于表示 Data Lake Store 或其中的文件夹（在 `source` 中指定）将在 DBFS 中装载的位置。
   * `dbutils.secrets.get(scope="<scope-name>",key="<service-credential-key-name>")` 检索作为[机密](../../../security/secrets/secrets.md)已存储在[机密范围](../../../security/secrets/secret-scopes.md)中的服务凭据。
2. 访问 Azure Data Lake Storage Gen2 文件系统中的文件，就像访问 DBFS 中的文件一样；例如：

   #### <a name="python"></a>Python

   ```python
   df = spark.read.text("/mnt/%s/...." % <mount-name>)
   df = spark.read.text("dbfs:/mnt/<mount-name>/....")
   ```

   #### <a name="scala"></a>Scala

   ```scala
   val df = spark.read.text("/mnt/<mount-name>/....")
   val df = spark.read.text("dbfs:/mnt/<mount-name>/....")
   ```

### <a name="unmount-a-mount-point"></a>卸载装入点

若要卸载装入点，请使用以下命令：

```python
dbutils.fs.unmount("/mnt/<mount-name>")
```

## <a name="access-directly-with-service-principal-and-oauth-20"></a><a id="access-directly-with-service-principal-and-oauth-20"> </a><a id="adls-gen2-oauth-2"> </a>直接使用服务主体和 OAuth 2.0 进行访问

可以使用服务主体通过 OAuth 2.0 直接访问 Azure Data Lake Storage Gen2 存储帐户（而不是使用 DBFS 装载）。 你可以直接访问服务主体有权访问的任何 Azure Data Lake Storage Gen2 存储帐户。 可以在同一 Spark 会话中添加多个存储帐户和服务主体。

### <a name="set-credentials"></a>设置凭据

设置凭据的方式取决于在访问 Azure Data Lake Storage Gen2 时计划使用的 API：数据帧、数据集或 RDD。

#### <a name="dataframe-or-dataset-api"></a>数据帧或数据集 API

如果你使用的是 Spark 数据帧或数据集 API，我们建议你在笔记本的会话配置中设置帐户凭据：

```scala
spark.conf.set("fs.azure.account.auth.type.<storage-account-name>.dfs.core.chinacloudapi.cn", "OAuth")
spark.conf.set("fs.azure.account.oauth.provider.type.<storage-account-name>.dfs.core.chinacloudapi.cn", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
spark.conf.set("fs.azure.account.oauth2.client.id.<storage-account-name>.dfs.core.chinacloudapi.cn", "<application-id>")
spark.conf.set("fs.azure.account.oauth2.client.secret.<storage-account-name>.dfs.core.chinacloudapi.cn", dbutils.secrets.get(scope="<scope-name>",key="<service-credential-key-name>"))
spark.conf.set("fs.azure.account.oauth2.client.endpoint.<storage-account-name>.dfs.core.chinacloudapi.cn", "https://login.microsoftonline.com/<directory-id>/oauth2/token")
```

其中，`dbutils.secrets.get(scope="<scope-name>",key="<service-credential-key-name>")` 用于检索已作为[机密](../../../security/secrets/secrets.md)存储在[机密范围](../../../security/secrets/secret-scopes.md)中的服务凭据。

#### <a name="rdd-api"></a>RDD API

如果使用 RDD API 访问 Azure Data Lake Storage Gen2，则无法使用 `spark.conf.set(...)` 访问 Hadoop 配置选项集。 因此，必须使用以下方法之一设置凭据：

* 创建群集时，将 Hadoop 配置选项指定为 Spark 选项。 必须将 `spark.hadoop.` 前缀添加到相应的 Hadoop 配置键，以便将它们传播到用于 RDD 作业的 Hadoop 配置：

  ```ini
  fs.azure.account.auth.type.<storage-account-name>.dfs.core.chinacloudapi.cn OAuth
  fs.azure.account.oauth.provider.type.<storage-account-name>.dfs.core.chinacloudapi.cn org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider
  fs.azure.account.oauth2.client.id.<storage-account-name>.dfs.core.chinacloudapi.cn <application-id>
  fs.azure.account.oauth2.client.secret.<storage-account-name>.dfs.core.chinacloudapi.cn <service-credential>
  fs.azure.account.oauth2.client.endpoint.<storage-account-name>.dfs.core.chinacloudapi.cn https://login.microsoftonline.com/<directory-id>/oauth2/token
  ```

* Scala 用户可在 `spark.sparkContext.hadoopConfiguration` 中设置凭据：

  ```scala
  spark.sparkContext.hadoopConfiguration.set("fs.azure.account.auth.type.<storage-account-name>.dfs.core.chinacloudapi.cn", "OAuth")
  spark.sparkContext.hadoopConfiguration.set("fs.azure.account.oauth.provider.type.<storage-account-name>.dfs.core.chinacloudapi.cn",  "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
  spark.sparkContext.hadoopConfiguration.set("fs.azure.account.oauth2.client.id.<storage-account-name>.dfs.core.chinacloudapi.cn", "<application-id>")
  spark.sparkContext.hadoopConfiguration.set("fs.azure.account.oauth2.client.secret.<storage-account-name>.dfs.core.chinacloudapi.cn", dbutils.secrets.get(scope="<scope-name>",key="<service-credential-key-name>"))
  spark.sparkContext.hadoopConfiguration.set("fs.azure.account.oauth2.client.endpoint.<storage-account-name>.dfs.core.chinacloudapi.cn", "https://login.microsoftonline.com/<directory-id>/oauth2/token")
  ```

  其中，`dbutils.secrets.get(scope="<scope-name>",key="<service-credential-key-name>")` 用于检索已作为[机密](../../../security/secrets/secrets.md)存储在[机密范围](../../../security/secrets/secret-scopes.md)中的服务凭据。

> [!WARNING]
>
> 这些凭据可供访问群集的所有用户使用。

设置凭据后，可以使用标准 Spark 和 Databricks API 从存储帐户读取。 例如： 。

```scala
val df = spark.read.parquet("abfss://<file-system-name>@<storage-account-name>.dfs.core.chinacloudapi.cn/<directory-name>")

dbutils.fs.ls("abfss://<file-system-name>@<storage-account-name>.dfs.core.chinacloudapi.cn/<directory-name>")
```

## <a name="access-directly-using-the-storage-account-access-key"></a><a id="access-directly-using-the-storage-account-access-key"> </a><a id="adls-gen2-access-key"> </a>直接使用存储帐户访问密钥进行访问

可使用存储帐户访问密钥访问 Azure Data Lake Storage Gen2 存储帐户。

### <a name="set-your-credentials"></a>设置凭据

设置凭据的方式取决于在访问 Azure Data Lake Storage Gen2 时计划使用的 API：数据帧、数据集或 RDD。

#### <a name="dataframe-or-dataset-api"></a>数据帧或数据集 API

如果你使用的是 Spark 数据帧或数据集 API，我们建议你在笔记本的会话配置中设置帐户凭据：

```scala
spark.conf.set(
  "fs.azure.account.key.<storage-account-name>.dfs.core.chinacloudapi.cn",
  dbutils.secrets.get(scope="<scope-name>",key="<storage-account-access-key-name>"))
```

其中 `dbutils.secrets.get(scope="<scope-name>",key="<storage-account-access-key-name>")` 用于检索存储帐户访问密钥，该密钥已作为[机密](../../../security/secrets/secrets.md)存储在[机密范围](../../../security/secrets/secret-scopes.md)中。

#### <a name="rdd-api"></a>RDD API

如果使用 RDD API 访问 Azure Data Lake Storage Gen2，则无法使用 `spark.conf.set(...)` 访问 Hadoop 配置选项集。 因此，必须使用以下方法之一设置凭据：

* 创建群集时，将 Hadoop 配置选项指定为 Spark 选项。 必须将 `spark.hadoop.` 前缀添加到相应的 Hadoop 配置键，以便将它们传播到用于 RDD 作业的 Hadoop 配置：

  ```ini
  # Using an account access key
  spark.hadoop.fs.azure.account.key.<storage-account-name>.dfs.core.chinacloudapi.cn <storage-account-access-key-name>
  ```

* Scala 用户可在 `spark.sparkContext.hadoopConfiguration` 中设置凭据：

  ```scala
  // Using an account access key
  spark.sparkContext.hadoopConfiguration.set(
    "fs.azure.account.key.<storage-account-name>.dfs.core.chinacloudapi.cn",
    dbutils.secrets.get(scope="<scope-name>",key="<storage-account-access-key-name>")
  )
  ```

  其中 `dbutils.secrets.get(scope="<scope-name>",key="<storage-account-access-key-name>")` 用于检索存储帐户访问密钥，该密钥已作为[机密](../../../security/secrets/secrets.md)存储在[机密范围](../../../security/secrets/secret-scopes.md)中。

> [!WARNING]
>
> 这些凭据可供访问群集的所有用户使用。

设置凭据后，可以使用标准 Spark 和 Databricks API 从存储帐户读取。 例如，

```scala
val df = spark.read.parquet("abfss://<file-system-name>@<storage-account-name>.dfs.core.chinacloudapi.cn/<directory-name>")

dbutils.fs.ls("abfss://<file-system-name>@<storage-account-name>.dfs.core.chinacloudapi.cn/<directory-name>")
```

以下笔记本演示直接使用装载操作访问 Azure Data Lake Storage Gen2。

##### <a name="adls-gen2-service-principal-notebook"></a>ADLS Gen2 服务主体笔记本

[获取笔记本](../../../_static/notebooks/adls-service-prin-gen2.html)

## <a name="frequently-asked-questions-faq"></a>常见问题 (FAQ)

**ABFS 是否支持共享访问签名 (SAS) 令牌身份验证**？

ABFS 不支持 SAS 令牌身份验证，但 Azure Data Lake Storage Gen2 服务本身确实支持 SAS 密钥。

**我可以使用 `abfs` 方案访问 Azure Data Lake Storage Gen2 吗？**

是的。 但建议尽可能使用 `abfss` 方案，该方案使用 SSL 加密访问。 需要在 OAuth 或基于 Azure Active Directory 的身份验证中使用 `abfss`，因为任何传递令牌的 Azure AD 方面自然都需要使用安全传输。

**当我访问启用了分层命名空间的 Azure Data Lake Storage Gen2 帐户时，我遇到了一个 `java.io.FileNotFoundException` 错误，并且错误消息包含 `FilesystemNotFound`** 。

如果错误消息包含以下信息，则这是因为你的命令正在尝试访问通过 Azure 门户创建的 Blob 存储容器：

```
StatusCode=404
StatusDescription=The specified filesystem does not exist.
ErrorCode=FilesystemNotFound
ErrorMessage=The specified filesystem does not exist.
```

启用分层命名空间后，不需要通过 Azure 门户创建容器。 如果看到此问题，请通过 Azure 门户删除 Blob 容器。 几分钟后，你就可以访问该容器。 或者，可以更改 `abfss` URI 以使用其他容器，只要此容器不是通过 Azure 门户创建的。

**我在尝试装载 Azure Data Lake Storage Gen2 系统文件时，观察到错误 `This request is not authorized to perform this operation using this permission`** 。

如果未授予用于 Azure Data Lake Storage Gen2 的服务主体适当的角色分配，则会发生此错误。 请参阅[使用 Azure Active Directory 凭据自动访问](#gen2-service-principal)。

## <a name="known-issues"></a>已知问题

请参阅 Microsoft 文档中的 [Azure Data Lake Storage Gen2 的已知问题](https://aka.ms/adlsgen2knownissues)。