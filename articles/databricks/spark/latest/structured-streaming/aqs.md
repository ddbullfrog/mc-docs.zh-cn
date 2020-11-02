---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/03/2020
title: 将优化的 Azure Blob 存储文件源与 Azure 队列存储配合使用 - Azure Databricks
description: 了解如何使用 Azure 存储（队列和 Blob）作为 Azure Databricks 中用于流式传输数据的源。
ms.openlocfilehash: b2f7b784677ac6c788d6171cbdb9e254b644e540
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472772"
---
# <a name="optimized-azure-blob-storage-file-source-with-azure-queue-storage"></a>将优化的 Azure Blob 存储文件源与 Azure 队列存储配合使用

Databricks ABS-AQS 连接器使用 Azure 队列存储 (AQS) 提供优化的文件源，允许你查找写入到 Azure Blob 存储 (ABS) 容器的新文件，而无需重复列出所有文件。 这有两个主要优点：

* 延迟较低：无需列出 ABS 上嵌套的目录结构，此列出操作速度很慢且会消耗大量资源。
* 成本较低：不再向 ABS 发出成本很高的 LIST API 请求。

> [!IMPORTANT]
>
> * 当事件被确认时，ABS-AQS 源将从 AQS 队列中删除消息。 如果要让其他管道使用同一队列中的内容，请为经过优化的读取器设置单独的 AQS 队列。 你可以设置多个可发布到不同队列的事件网格订阅。
> * ABS-AQS 源仅支持加载存储在 Azure Blob 存储中的文件。 如果使用 [Azure Data Lake Storage Gen1](https://docs.microsoft.com/azure/data-lake-store/) 或 [Azure Data Lake Storage Gen2](/storage/data-lake-storage/introduction)，建议你使用[自动加载程序](auto-loader.md)。

## <a name="use-the-abs-aqs-file-source"></a>使用 ABS-AQS 文件源

若要使用 ABS-AQS 文件源，必须执行以下操作：

* 使用 Azure 事件网格订阅设置 ABS 事件通知，并将其路由到 AQS。 参阅[对 Blob 存储事件做出反应](/storage/blobs/storage-blob-event-overview)。
* 指定 `fileFormat` 和 `queueUrl` 选项以及架构。 例如： 。

  ```python
  spark.readStream \
    .format("abs-aqs") \
    .option("fileFormat", "json") \
    .option("queueName", ...) \
    .option("connectionString", ...) \
    .schema(...) \
    .load()
  ```

## <a name="authenticate-with-azure-queue-storage-and-blob-storage"></a>使用 Azure 队列存储和 Blob 存储进行身份验证

若要使用 Azure 队列存储和 Blob 存储进行身份验证，需要使用共享访问签名 (SAS) 令牌或存储帐户密钥。 需要为在其中部署了队列的存储帐户提供一个连接字符串，其中会包含你的存储帐户的 SAS 令牌或访问密钥。 有关详细信息，请参阅[配置 Azure 存储连接字符串](/storage/common/storage-configure-connection-string)。

你还需要提供对 Azure Blob 存储容器的访问权限。 若要了解如何配置对 Azure Blob 存储容器的访问权限，请参阅 [Azure Blob 存储](../../../data/data-sources/azure/azure-storage.md#azure-storage)。

> [!NOTE]
>
> 强烈建议你使用[机密](../../../security/secrets/secrets.md)来提供连接字符串。

## <a name="configuration"></a>配置

| 选项                    | 类型                                                                  | 默认                      | 说明                                                                                                                                                                                                                                                                                                                                                                                                                  |
|---------------------------|-----------------------------------------------------------------------|------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| queueName                 | 字符串                                                                | 无（必需参数）        | AQS 队列的名称。                                                                                                                                                                                                                                                                                                                                                                                                   |
| fileFormat                | 字符串                                                                | 无（必需参数）        | 文件的格式，例如 `parquet`、`json`、`csv`、`text`，等等。                                                                                                                                                                                                                                                                                                                                                 |
| connectionString          | 字符串                                                                | 无（必需参数）        | 用来访问队列的[连接字符串](/storage/common/storage-configure-connection-string)。                                                                                                                                                                                                                                                                                           |
| queueFetchInterval        | 持续时间字符串，例如，`2m` 表示 2 分钟。                   | `"5s"`                       | 当队列为空时，在两次提取之间等待的时长。 Azure 按照向 AQS 发出的 API 请求收费。 因此，如果数据不是频繁到达，则可以将此值设置为较长的持续时间。 只要队列不为空，我们就会连续提取。 如果每 5 分钟创建一次新文件，则可能需要设置较高的 `queueFetchInterval` 以降低 AQS 成本。                                                      |
| pathRewrites              | JSON 字符串。                                                        | `"{}"`                       | 如果使用装入点，则可以使用装入点重写 `container@storageAccount/key` 路径的前缀。 只能重写前缀。 例如，对于配置 `{"myContainer@myStorageAccount/path": "dbfs:/mnt/data-warehouse"}`，路径 `wasbs://myContainer@myStorageAccount.blob.windows.core.net/path/2017/08/fileA.json` 被重写为<br>`dbfs:/mnt/data-warehouse/2017/08/fileA.json`。 |
| ignoreFileDeletion        | 布尔                                                               | `false`                      | 如果你进行了生命周期配置或手动删除了源文件，则必须将此选项设置为 `true`。                                                                                                                                                                                                                                                                                                            |
| maxFileAge                | Integer                                                               | 604800                       | 确定将文件通知作为状态存储多长时间（以秒为单位）以防止重复处理。                                                                                                                                                                                                                                                                                                                     |
| allowOverwrites           | 布尔                                                               | `true`                       | 是否应重新处理重写的 blob。                                                                                                                                                                                                                                                                                                                                                                  |

如果在驱动程序日志中看到类似于 `Fetched 0 new events and 3 old events.` 的大量消息，则在这种情况下，你通常会看到比新事件更多的旧事件，应缩短流的触发间隔。

如果要从 Blob 存储中的某个位置使用文件，而这些文件可能会在处理之前被删除，则可以设置以下配置以忽略错误并继续处理：

```python
spark.sql("SET spark.sql.files.ignoreMissingFiles=true")
```

## <a name="frequently-asked-questions-faq"></a>常见问题 (FAQ)

**如果 `ignoreFileDeletion` 为 False（默认值），并且对象已被删除，这是否会导致整个管道故障？**

是的，如果收到一个事件，指出该文件已被删除，则会导致整个管道故障。

**应当如何设置 `maxFileAge`？**

Azure 队列存储提供“至少一次”消息传送语义，因此，我们需要保留状态以进行重复数据删除。 `maxFileAge` 的默认设置为 7 天，这等于队列中消息的最大 TTL。