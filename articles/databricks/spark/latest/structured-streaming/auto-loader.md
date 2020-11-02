---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/02/2020
title: 使用自动加载程序从 Azure Blob 存储或 Azure Data Lake Storage Gen2 加载文件 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用自动加载程序从 Azure Blob 存储或 Azure Data Lake Storage Gen2 引入数据。
ms.openlocfilehash: 825f085ee0b3c785d2ad09c26f6ffbc5cc8885e4
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472984"
---
# <a name="load-files-from-azure-blob-storage-or-azure-data-lake-storage-gen2-using-auto-loader"></a>使用自动加载程序从 Azure Blob 存储或 Azure Data Lake Storage Gen2 加载文件

自动加载程序会在新数据文件到达 Azure Blob 存储或 Azure Data Lake Storage Gen2 时以增量方式高效地对其进行处理，无需进行任何其他设置。 自动加载程序提供了名为 `cloudFiles` 的新的结构化流式处理源。 给定云文件存储上的输入目录路径后，`cloudFiles` 源将在新文件到达时自动处理这些文件，你也可以选择处理该目录中的现有文件。

## <a name="requirements"></a>要求

Databricks Runtime 7.2 或更高版本。

如果使用 Databricks Runtime 7.1 或更低版本创建了流，请参阅[默认选项值和兼容性的更改](#compatibility)和[云资源管理](#cloud-resource-management)。

## <a name="new-file-detection-modes"></a>新文件检测模式

当存在新文件时，自动加载程序支持两种检测模式：

* **目录列表** ：按输入目录的并行列表识别新文件。 目录列表模式允许你快速启动自动加载程序流，无需进行任何权限配置，适用于仅需定期将少数文件流式传输进来的情况。 目录列表模式是 Databricks Runtime 7.2 及更高版本中自动加载程序的默认设置。

在 Databricks Runtime 7.3 及更高版本中，自动加载程序在目录列表模式中还支持 Azure Data Lake Storage Gen 1。

* **文件通知** ：使用从输入目录订阅文件事件的 Azure 事件网格和队列存储服务。 自动加载程序会自动设置 Azure 事件网格和队列存储服务。 对于大型输入目录，文件通知模式的性能和可伸缩性更高。 若要使用此模式，你必须为 Azure 事件网格和队列存储服务配置[权限](#permissions)，并指定

  ```python
  .option("cloudFiles.useNotifications", "true")
  ```

你可以在重启流时更改模式。 例如，当目录列表由于输入目录大小增加而变得太慢时，你可能需要切换到文件通知模式。 对于这两种模式，自动加载程序会在内部跟踪哪些文件已处理以提供“只执行一次”语义，因此你无需自己管理任何状态信息。

## <a name="use-cloudfiles-source"></a>使用 `cloudFiles` 源

使用 `cloudFiles` 源的方式与其他流式处理源相同：

### <a name="python"></a>Python

```python
df = spark.readStream.format("cloudFiles") \
  .option(<cloudFiles-option>, <option-value>) \
  .schema(<schema>) \
  .load(<input-path>)

df.writeStream.format("delta") \
  .option("checkpointLocation", <checkpoint-path>) \
  .start(<output-path>)
```

### <a name="scala"></a>Scala

```scala
val df = spark.readStream.format("cloudFiles")
  .option(<cloudFiles-option>, <option-value>)
  .schema(<schema>)
  .load(<input-path>)

df.writeStream.format("delta")
  .option("checkpointLocation", <checkpoint-path>)
  .start(<output-path>)
```

其中：

* `<cloudFiles-option>` 是一个选项，`<option-value>` 是[配置](#configuration)中列出的选项值。
* `<schema>` 是文件架构。
  .. 注意：在 Databricks Runtime 7.3 及更高版本中，如果文件格式为 `text` 或 `binaryFile`，则无需提供架构。
* `<input-path>` 是 Azure Blob 存储或 Azure Data Lake Storage Gen2 中用于监视新文件的路径。 还会监视 `<input-path>` 的子目录。 `<input-path>` 可以包含文件 glob 模式。
* `<checkpoint-path>` 是输出流检查点位置。
* `<output-path>` 是输出流路径。

## <a name="configuration"></a>配置

特定于 `cloudFiles` 源的配置选项以 `cloudFiles` 为前缀，因此它们位于与其他结构化流式处理源选项不同的命名空间中。

> [!IMPORTANT]
>
> 某些默认选项值在 Databricks Runtime 7.2 中已更改。 如果你使用的是 Databricks Runtime 7.1 或更低版本中的自动加载程序，请参阅[默认选项值和兼容性的更改](#compatibility)以获取详细信息。

| 选项                           | 类型         | 默认                  | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|----------------------------------|--------------|--------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| cloudFiles.format                | 字符串       | 无（必需选项）   | 源路径中的数据文件格式。 `json`、`csv`、`text`、`parquet`、`binaryFiles`，等等。                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| cloudFiles. includeExistingFiles | 布尔      | 是                     | 是否在流式处理中包含输入路径中的现有文件，而不是仅处理在设置通知后到达的新文件。 仅在首次启动流时会考虑此选项。 在流重启时更改其值不会产生任何效果。                                                                                                                                                                                                                                             |
| cloudFiles. maxFilesPerTrigger   | Integer      | 1000                     | 要在每个触发器中处理的最大新文件数。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| cloudFiles. maxBytesPerTrigger   | 字节字符串  | 无                     | 要在每个触发器中处理的最大新字节数。 你可以指定一个字节字符串（例如 `10g`），将每个微批限制为 10 GB 数据。 这个一个软性最大值。 如果每个文件为 3 GB，则 Azure Databricks 在一个微批中可以处理 12 GB。 与 `cloudFiles.maxFilesPerTrigger` 一起使用时，Azure Databricks 最多将消耗<br>`cloudFiles.maxFilesPerTrigger` 或 `cloudFiles.maxBytesPerTrigger` 的下限，以先达到者为准。 与 `Trigger.Once()` 一起使用时，此选项不起作用。 |
| cloudFiles. useNotifications     | Boolean      | false                    | 是否使用文件通知模式来确定何时存在新文件。 如果为 false，将使用目录列表模式。                                                                                                                                                                                                                                                                                                                                                                                                                             |
| cloudFiles. validateOptions      | 布尔      | 是                     | 是否验证自动加载程序选项，并为未知或不一致的选项返回错误。                                                                                                                                                                                                                                                                                                                                                                                                                                               |

只有当选择了文件通知模式 (`cloudFiles.useNotifications` = `true`) 时，才需要提供以下身份验证选项：

| 身份验证选项         | 类型         | 默认                  | 说明                                                                                                                                                                                                                                                                           |
|-------------------------------|--------------|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| cloudFiles.connectionString   | 字符串       | 无                     | 存储帐户的连接字符串，基于帐户访问密钥或共享访问签名 (SAS)。                                                                                                                                                                   |
| cloudFiles.resourceGroup      | 字符串       | 无                     | 在其下创建了存储帐户的 Azure 资源组。                                                                                                                                                                                                                  |
| cloudFiles.subscriptionId     | 字符串       | 无                     | 在其下创建了资源组的 Azure 订阅 ID。                                                                                                                                                                                                                  |
| cloudFiles.tenantId           | 字符串       | 无                     | 在其下创建了服务主体的 Azure 租户 ID。                                                                                                                                                                                                                     |
| cloudFiles.clientId           | 字符串       | 无                     | 服务主体的客户端 ID 或应用程序 ID。                                                                                                                                                                                                                            |
| cloudFiles.clientSecret       | 字符串       | 无                     | 服务主体的客户端密码。                                                                                                                                                                                                                                           |
| cloudFiles.queueName          | 字符串       | 无                     | Azure 队列的 URL。 如果提供了此项，则云文件源会直接使用此队列中的事件，而不是设置自己的 Azure 事件网格和队列存储服务。  在这种情况下，你的<br>`cloudFiles.connectionString` 只需要对队列具有读取权限。 |

### <a name="changes-in-default-option-values-and-compatibility"></a><a id="changes-in-default-option-values-and-compatibility"> </a><a id="compatibility"> </a>默认选项值和兼容性的更改

Databricks Runtime 7.2 中以下自动加载程序选项的默认值已更改为[配置](#configuration)中列出的值。

* `cloudFiles.useNotifications`
* `cloudFiles.includeExistingFiles`
* `cloudFiles.validateOptions`

在 Databricks Runtime 7.1 及更低版本中启动的自动加载程序流具有以下默认选项值：

* `cloudFiles.useNotifications` 为 `true`
* `cloudFiles.includeExistingFiles` 为 `false`
* `cloudFiles.validateOptions` 为 `false`

为了确保与现有应用程序的兼容性，当你在 Databricks Runtime 7.2 或更高版本上运行现有的自动加载程序流时，这些默认选项值不会更改；这些流在升级后将具有相同的行为。

## <a name="permissions"></a>权限

你必须具有对输入目录的读取权限。  请参阅 [Azure Blob 存储](../../../data/data-sources/azure/azure-storage.md)和 [Azure Data Lake Gen2](../../../data/data-sources/azure/azure-datalake-gen2.md)。

若要使用文件通知模式，你必须提供两组凭据：

* 连接字符串

  自动加载程序需要使用[连接字符串](/storage/common/storage-configure-connection-string)对 Azure 队列存储操作（例如，创建队列以及从队列中读取和删除消息）进行身份验证。 队列是在输入目录路径所在的同一存储帐户中创建的。
  你可以在[帐户密钥](/storage/common/storage-account-keys-manage)或[共享访问签名 (SAS)](/storage/common/storage-sas-overview) 中找到连接字符串。 配置 SAS 令牌时，必须提供以下权限：

  > [!div class="mx-imgBorder"]
  > ![自动加载程序权限](../../../_static/images/spark/structured-streaming/auto-loader-permissions.png)

* 服务主体

  自动加载程序要求服务主体采用客户端 ID 和客户端密码的形式，以便从输入路径所在的存储帐户配置事件网格和事件通知。 你必须创建 [Azure Active Directory 应用和服务主体](/active-directory/develop/howto-create-service-principal-portal)。 除了[事件网格权限](/event-grid/security-authorization)之外，你还必须为此应用分配你的输入路径所在的存储帐户的“参与者”角色。

## <a name="troubleshooting"></a>疑难解答

**错误：`java.lang.RuntimeException: Failed to create event grid subscription.`**

如果你第一次运行自动加载程序时看到此错误消息，则很可能是因为事件网格未在 Azure 订阅中注册为资源提供程序。 若要在 Azure 门户中注册它，请执行以下操作：

1. 转到你的订阅。
2. 单击“设置”部分的“资源提供程序”。
3. 注册提供程序 `Microsoft.EventGrid`。

**错误：`403 Forbidden ... does not have authorization to perform action 'Microsoft.EventGrid/eventSubscriptions/[read|write]' over scope ...`**

  如果你第一次运行自动加载程序时看到此错误消息，请确保已向你的事件网格和存储帐户的服务主体授予了“参与者”角色。

## <a name="cloud-resource-management"></a>云资源管理

可以使用 Scala API 管理由自动加载程序创建的 Azure 事件网格和队列存储服务。 使用此 API 之前，必须配置[权限](#permissions)中所述的资源设置权限。

```scala
import com.databricks.sql.CloudFilesAzureResourceManager
val manager = CloudFilesAzureResourceManager
  .newManager
  .option("cloudFiles.connectionString", <connection-string>)
  .option("cloudFiles.resourceGroup", <resource-group>)
  .option("cloudFiles.subscriptionId", <subscription-id>)
  .option("cloudFiles.tenantId", <tenant-id>)
  .option("cloudFiles.clientId", <service-principal-client-id>)
  .option("cloudFiles.clientSecret", <service-principal-client-secret>)
  .create()

// List notification services created by Auto Loader
manager.listNotificationServices()

// Tear down the notification services created for a specific stream ID.
// Stream ID is a GUID string that you can find in the list result above.
manager.tearDownNotificationServices(<stream-id>)
```

## <a name="frequently-asked-questions-faq"></a>常见问题 (FAQ)

**是否需要事先创建 Azure 事件通知服务？**

否。 如果你选择了文件通知模式，则自动加载程序会在你启动流时自动创建 Azure Blob 存储或 Azure Data Lake Storage Gen2 > 事件网格订阅 > 队列文件事件通知管道。

**如何清除由自动加载程序创建的事件通知资源（例如事件网格订阅和队列）？**

目前，必须通过 Web 门户或 API 手动删除这些资源。 自动加载程序创建的所有资源都有以下前缀：`databricks-`。

**当文件被追加或覆盖时，自动加载程序是否会再次处理文件？**

否。 文件只会处理一次。 如果文件被追加或覆盖，Azure Databricks 不保证处理的是哪个版本的文件。 对于明确定义的行为，建议使用自动加载程序来仅引入不可变的文件。 如果这不能满足你的要求，请联系你的 Databricks 代表。

**能否从同一输入目录运行多个流式处理查询？**

是的。 每个云文件流（由唯一的检查点目录标识）都有自己的队列，同一 Azure Blob 存储或 Azure Data Lake Storage Gen2 事件可以发送到多个队列。

**如果我的数据文件没有连续送达，但会定期送达（例如，一天一次），那么我是否仍然可以使用此源，是否有任何好处？**

是的，的确有好处。 在这种情况下，你可以设置一个 `Trigger-Once` 结构化流式处理作业，对其进行计划，使之在预计的文件到达时间之后运行。 第一次运行会设置事件通知服务，该服务将始终打开，即使流式处理群集处于关闭状态也是如此。 重启流时，`cloudFiles` 源将提取并处理在队列中备份的所有文件事件。 在这种情况下使用自动加载程序的好处是，你无需确定哪些文件是新的并且每次都对其进行处理，那样做的成本会非常昂贵。

**如果在重启流时更改了检查点位置，会发生什么情况？**

检查点位置维护流的重要标识信息。 更改检查点位置实际上意味着已放弃上一个流并启动一个新流。 新的流将创建新的进度信息，如果使用的是文件通知模式，则可以使用新的 Azure 事件网格和队列存储服务。 必须为任何已放弃的流手动清理检查点位置、Azure 事件网格和队列存储服务。