---
title: Kusto.Ingest 错误和异常 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 Kusto.Ingest - 错误和异常。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 10/30/2019
ms.date: 10/29/2020
ms.openlocfilehash: 289b21997a3210b6ad8f2a3a0528fd949430f7c3
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105324"
---
# <a name="kustoingest-errors-and-exceptions"></a>Kusto.Ingest 错误和异常
在客户端上处理引入时发生的任何错误都通过 C# 异常来指示。

## <a name="failures"></a>失败数

### <a name="kustodirectingestclient-exceptions"></a>KustoDirectIngestClient 异常

尝试从多个源进行引入时，在引入过程中可能会发生错误。 如果某个源的引入失败，则会记录该失败，客户端会继续引入剩余的源。 完成用于引入的所有源后，会引发 `IngestClientAggregateException`，其中包含 `IList<IngestClientException> IngestionErrors` 成员。

`IngestClientException` 及其派生类包含 `IngestionSource` 字段和 `Error` 字段。 这两个字段共同创建了一个映射，从引入失败的源映射到尝试引入时发生的错误。 此信息可用于 `IngestionErrors` 列表，用以调查哪些源引入失败以及原因是什么。 `IngestClientAggregateException` 异常还包含布尔属性 `GlobalError`，用于指示是否所有源都发生了某个错误。

### <a name="failures-ingesting-from-files-or-blobs"></a>从文件或 blob 引入时的失败

如果尝试从 blob 或文件引入时发生引入失败，则即使 `deleteSourceOnSuccess` 标志设置为 `true`，也不会删除引入源。 将保留源供进一步分析。 在了解错误来源后，如果该错误不是源自引入源本身，则客户端的用户可以尝试重新引入它。

### <a name="failures-ingesting-from-idatareader"></a>从 IDataReader 引入时的失败

从 DataReader 引入时，要引入的数据将保存到默认位置为 `<Temp Path>\Ingestions_<current date and time>` 的一个临时文件夹中。 成功引入后，始终会删除此默认文件夹。

在 `IngestFromDataReader` 和 `IngestFromDataReaderAsync` 方法中，`retainCsvOnFailure` 标志（其默认值为 `false`）决定了引入失败后是否应保留文件。 如果此标志设置为 `false`，则不会持久保存引入失败的数据，因此很难了解发生了什么问题。

## <a name="kustoqueuedingestclient-exceptions"></a>KustoQueuedIngestClient 异常

`KustoQueuedIngestClient` 通过将消息上传到 Azure 队列来引入数据。 如果在排队过程之前或排队过程中发生错误，则在该过程结束时会引发 `IngestClientAggregateException`。 引发的异常包括一个 `IngestClientException` 集合，该集合包含每个失败的源且尚未发布到队列。 还会引发尝试发布消息时发生的错误。

### <a name="posting-to-queue-failures-with-a-file-or-blob-as-a-source"></a>以文件或 blob 作为源发布到队列时的失败

如果使用 `KustoQueuedIngestClient` 的 `IngestFromFile/IngestFromBlob` 方法时发生错误，则即使 `deleteSourceOnSuccess` 标志设置为 `true`，也不会删除源。 相反，将保留源供进一步分析。 

在了解错误来源后，如果该错误不是源自引入源本身，则客户端的用户可以通过对失败的源使用相关的 `IngestFromFile/IngestFromBlob` 方法来尝试将数据重新排队。 

### <a name="posting-to-queue-failures-with-idatareader-as-a-source"></a>使用 IDataReader 作为源发布到队列时的失败

使用 DataReader 源时，发布到队列的数据将保存到默认位置为 `<Temp Path>\Ingestions_<current date and time>` 的一个临时文件夹中。 将数据成功发布到队列后，始终会删除此文件夹。
在 `IngestFromDataReader` 和 `IngestFromDataReaderAsync` 方法中，`retainCsvOnFailure` 标志（其默认值为 `false`）决定了引入失败后是否应保留文件。 如果此标志设置为 `false`，则不会持久保存引入失败的数据，因此很难了解发生了什么问题。

### <a name="common-failures"></a>常见失败
|错误                         |Reason           |缓解措施                                   |
|------------------------------|-----------------|---------------------------------------------|
|数据库 <database name> 名称不存在| 数据库不存在|在 `kustoIngestionProperties` 中检查数据库名称/创建数据库 |
|找不到类型为 "Table" 的实体“不存在的表名”。|该表不存在，也没有 CSV 映射。| 添加 CSV 映射/创建所需的表 |
|Blob <blob path> 被排除，原因如下：必须通过 jsonMapping 参数引入 JSON 模式| 进行 JSON 引入时未提供 JSON 映射。|提供 JSON 映射 |
|下载 blob 失败：“远程服务器返回了错误: (404)未找到。”| Blob 不存在。|请验证该 blob 是否存在。 如果它存在，请重试并联系 Kusto 团队 |
|JSON 列映射无效：两个或多个映射元素指向同一列。| JSON 映射有 2 个不同路径的列|纠正 JSON 映射 |
|EngineError - [UtilsException] `IngestionDownloader.Download`：一个或多个文件下载失败（请在 KustoLogs 中搜索 ActivityID:<GUID1> 和 RootActivityId:<GUID2>）| 一个或多个文件下载失败。 |重试 |
|无法分析:ID 为“<stream name>”的流的 CSV 格式错误，无法通过 ValidationOptions 策略 |CSV 文件格式不正确（例如，每行的列数不相同）。 仅当验证策略设置为 `ValidationOptions` 时才会失败。 ValidateCsvInputConstantColumns |检查你的 CSV 文件。 此消息仅适用于 CSV/TSV 文件 |
|`IngestClientAggregateException`，错误消息为“缺少有效共享访问签名的必需参数” |使用的 SAS 是服务的，而不是存储帐户的 |使用存储帐户的 SAS |

### <a name="ingestion-error-codes"></a>引入错误代码

为了便于以编程方式处理引入失败，我们对失败消息进行了扩充，在其中包含了数值错误代码 (`IngestionErrorCode enumeration`)。

|ErrorCode                                      |Reason                                                        |
|-----------------------------------------------|--------------------------------------------------------------|
|未知                                        | 发生未知错误|
|Stream_LowMemoryCondition                      | 操作用尽了内存|
|Stream_WrongNumberOfFields                     | CSV 文档包含不一致的字段数|
|Stream_InputStreamTooLarge                     | 提交的用于引入的文档超出了允许的大小|
|Stream_NoDataToIngest                          | 找不到要引入的数据流|
|Stream_DynamicPropertyBagTooLarge              | 引入的数据中的某个动态列包含太多的唯一属性|
|Download_SourceNotFound                        | 从 Azure 存储下载源失败 - 未找到源|
|Download_AccessConditionNotSatisfied           | 从 Azure 存储下载源失败 - 访问被拒绝|
|Download_Forbidden                             | 从 Azure 存储下载源失败 - 不允许访问|
|Download_AccountNotFound                       | 从 Azure 存储下载源失败 - 未找到帐户|
|Download_BadRequest                            | 从 Azure 存储下载源失败 - 请求不正确|
|Download_NotTransient                          | 从 Azure 存储下载源失败 - 非暂时性错误|
|Download_UnknownError                          | 从 Azure 存储下载源失败 - 未知错误|
|UpdatePolicy_QuerySchemaDoesNotMatchTableSchema| 调用更新策略失败。 查询架构与表架构不匹配|
|UpdatePolicy_FailedDescendantTransaction       | 调用更新策略失败。 后代事务性更新策略失败|
|UpdatePolicy_IngestionError                    | 调用更新策略失败。 发生引入错误|
|UpdatePolicy_UnknownError                      | 调用更新策略失败。 发生未知错误|
|BadRequest_MissingJsonMappingtFailure          | 没有使用 jsonMapping 参数引入 JSON 模式|
|BadRequest_InvalidBlob                         | 引擎无法打开和读取非 zip blob|
|BadRequest_EmptyBlob                           | 空 blob|
|BadRequest_EmptyArchive                        | zip 文件不包含任何已存档的元素|
|BadRequest_EmptyBlobUri                        | 指定的 blob URI 为空|
|BadRequest_DatabaseNotExist                    | 数据库不存在|
|BadRequest_TableNotExist                       | 表不存在|
|BadRequest_InvalidKustoIdentityToken           | Kusto 标识令牌无效|
|BadRequest_UriMissingSas                       | 源自未知 Blob 存储的 Blob 路径未包含 SAS|
|BadRequest_FileTooLarge                        | 尝试引入的文件太大|
|BadRequest_NoValidResponseFromEngine           | 未收到引入命令的有效回复|
|BadRequest_TableAccessDenied                   | 被拒绝访问表|
|BadRequest_MessageExhausted                    | 消息已用尽|
|General_BadRequest                             | 常规错误请求。 在引入到不存在的数据库/表时可能会提示|
|General_InternalServerError                    | 发生了内部服务器错误|

## <a name="detailed-exceptions-reference"></a>详细的异常参考信息

### <a name="cloudqueuesnotfoundexception"></a>CloudQueuesNotFoundException

当数据管理群集未返回任何队列时引发

基类：[异常](https://msdn.microsoft.com/library/system.exception(v=vs.110).aspx)

|字段名称 |类型     |含义
|-----------|---------|------------------------------|
|错误      | String  | 尝试从 DM 检索队列时发生的错误
                            
仅当使用 [Kusto 排队引入客户端](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient)时才适用。
在引入过程中，会多次尝试检索链接到 DM 的 Azure 队列。 当这些尝试失败时，将在“错误”字段中引发包含失败原因的异常。 还可能会在“InnerException”字段中引发内部异常。


### <a name="cloudblobcontainersnotfoundexception"></a>CloudBlobContainersNotFoundException

当数据管理群集未返回任何 blob 容器时引发

基类：[异常](https://msdn.microsoft.com/library/system.exception(v=vs.110).aspx)

|字段名称   |类型     |含义       
|-------------|---------|------------------------------|
|KustoEndpoint| String  | 相关 DM 的终结点
                            
仅当使用 [Kusto 排队引入客户端](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient)时才适用。  
当引入尚未存在于 Azure 容器中的源（例如文件、DataReader 或流）时，数据将上传到临时 blob 进行引入。 如果未找到要将数据上传到其中的容器，则会引发异常。

### <a name="duplicateingestionpropertyexception"></a>DuplicateIngestionPropertyException

在多次配置了某个引入属性时引发

基类：[异常](https://msdn.microsoft.com/library/system.exception(v=vs.110).aspx)

|字段名称   |类型     |含义       
|-------------|---------|------------------------------------|
|PropertyName | String  | 重复属性的名称
                            
### <a name="postmessagetoqueuefailedexception"></a>PostMessageToQueueFailedException

将消息发布到队列失败时引发

基类：[异常](https://msdn.microsoft.com/library/system.exception(v=vs.110).aspx)

|字段名称   |类型     |含义       
|-------------|---------|---------------------------------|
|QueueUri     | String  | 队列的 URI
|错误        | String  | 尝试发布到队列时生成的错误消息
                            
仅当使用 [Kusto 排队引入客户端](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient)时才适用。  
排队引入客户端通过将消息上传到相关的 Azure 队列来引入数据。 如果出现发布失败，则会引发异常。 它将包含队列 URI，在“错误”字段中包含失败原因，并可能在“InnerException”字段中包含内部异常。

### <a name="dataformatnotspecifiedexception"></a>DataFormatNotSpecifiedException

当需要指定数据格式但未在 `IngestionProperties` 中指定时引发

基类：IngestClientException

从流引入时，必须在 [IngestionProperties](kusto-ingest-client-reference.md#class-kustoingestionproperties) 中指定数据格式，以正确地引入数据。 当未指定 `IngestionProperties.Format` 时，将引发此异常。

### <a name="invaliduriingestclientexception"></a>InvalidUriIngestClientException

在将无效的 blob URI 作为引入源提交时引发

基类：IngestClientException

### <a name="compressfileingestclientexception"></a>CompressFileIngestClientException

当引入客户端无法压缩为引入提供的文件时引发

基类：IngestClientException

文件在引入之前会进行压缩。 当尝试压缩文件失败时，将引发异常。

### <a name="uploadfiletotempblobingestclientexception"></a>UploadFileToTempBlobIngestClientException

当引入客户端将为引入提供的源上传到临时 blob 失败时引发

基类：IngestClientException

### <a name="sizelimitexceededingestclientexception"></a>SizeLimitExceededIngestClientException

当引入源太大时引发

基类：IngestClientException

|字段名称   |类型     |含义       
|-------------|---------|-----------------------|
|大小         | long    | 引入源的大小
|MaxSize      | long    | 允许引入的最大大小

如果引入源超过最大大小 (4GB)，则会引发异常。 可以通过 [IngestionProperties 类](kusto-ingest-client-reference.md#class-kustoingestionproperties)中的 `IgnoreSizeLimit` 标志重写大小验证。 但是，建议不要[引入大于 1 GB 的单个源](about-kusto-ingest.md#ingestion-best-practices)。

### <a name="uploadfiletotempblobingestclientexception"></a>UploadFileToTempBlobIngestClientException

当引入客户端将为引入提供的文件上传到临时 blob 失败时引发

基类：IngestClientException

### <a name="directingestclientexception"></a>DirectIngestClientException

在执行直接引入的过程中出现常规错误时引发

基类：IngestClientException

### <a name="queuedingestclientexception"></a>QueuedIngestClientException

在执行排队引入的过程中发生错误时引发

基类：IngestClientException

### <a name="ingestclientaggregateexception"></a>IngestClientAggregateException

引入期间出现一个或多个错误时引发

基类：[AggregateException](https://msdn.microsoft.com/library/system.aggregateexception(v=vs.110).aspx)

|字段名称      |类型                             |含义       
|----------------|---------------------------------|-----------------------|
|IngestionErrors | IList<IngestClientException>    | 尝试引入时出现的错误，以及与之相关的源
|IsGlobalError   | bool                            | 指示是否所有源都发生了此异常

## <a name="next-steps"></a>后续步骤

有关原生代码中的错误的详细信息，请参阅[原生代码中的错误](../../concepts/errorsinnativecode.md)。
