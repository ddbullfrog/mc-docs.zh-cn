---
title: 在不使用 Ingest 库的情况下进行 Kusto 数据引入 - Azure 数据资源管理器
description: 本文介绍了如何在不使用 Azure 数据资源管理器中的 Kusto.Ingest 库的情况下进行数据引入。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
origin.date: 02/19/2020
ms.date: 10/29/2020
ms.openlocfilehash: b22b31dc040ada13a58bac3c1552e721e5f849b0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103571"
---
# <a name="ingestion-without-kustoingest-library"></a>在不使用 Kusto.Ingest 库的情况下进行引入

Kusto.Ingest 库是用于将数据引入到 Azure 数据资源管理器的首选方式。 不过，你还可以在不依赖于 Kusto.Ingest 包的情况下实现几乎相同的功能。
本文介绍如何使用目标为 Azure 数据资源管理器的针对生产级管道的 *排队引入* 来这样做。

> [!NOTE]
> 下面的代码是用 C# 编写的，并使用 Azure 存储 SDK、ADAL 身份验证库和 NewtonSoft.JSON 包来简化示例代码。 如果需要，可以将相应的代码替换为适当的 [Azure 存储 REST API](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api) 调用、[非 .NET ADAL 包](https://docs.azure.cn/active-directory/develop/active-directory-authentication-libraries)和任何可用的 JSON 处理包。

本文讲述的是建议的引入模式。 对于 Kusto.Ingest 库，其相应的实体是 [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) 接口。 这里，客户端代码通过将引入通知消息发布到 Azure 队列与 Azure 数据资源管理器服务交互。 对消息的引用是从 Kusto 数据管理（也称为引入）服务获取的。 与该服务的交互必须通过 Azure Active Directory (Azure AD) 进行身份验证。

下面的代码展示了 Kusto 数据管理服务如何在不使用 Kusto.Ingest 库的情况下处理排队数据引入。 如果完整的 .NET 由于环境或其他限制而无法访问或不可用，则此示例可能会很有用。

此代码包括了创建 Azure 存储客户端并将数据上传到 blob 的步骤。
在示例代码之后更详细地介绍了每个步骤。

1. [获取用于访问 Azure 数据资源管理器引入服务的身份验证令牌](#obtain-authentication-evidence-from-azure-ad)
1. 查询 Azure 数据资源管理器引入服务以获取：
    * [引入资源（队列和 blob 容器）](#retrieve-azure-data-explorer-ingestion-resources)
    * [将添加到每个引入消息中的 Kusto 标识令牌](#obtain-a-kusto-identity-token)
1. [将数据上传到在第 (2) 步从 Kusto 获取的某个 blob 容器中的 blob](#upload-data-to-the-azure-blob-container)
1. [撰写一条引入消息，用于标识目标数据库和表并指向第 (3) 步中的 blob](#compose-the-azure-data-explorer-ingestion-message)
1. [将在第 (4) 步撰写的引入消息发布到在第 (2) 步从 Azure 数据资源管理器获取的引入队列](#post-the-azure-data-explorer-ingestion-message-to-the-azure-data-explorer-ingestion-queue)**
1. [检索服务在引入过程中发现的任何错误](#check-for-error-messages-from-the-azure-queue)

```csharp
// A container class for ingestion resources we are going to obtain from Azure Data Explorer
internal class IngestionResourcesSnapshot
{
    public IList<string> IngestionQueues { get; set; } = new List<string>();
    public IList<string> TempStorageContainers { get; set; } = new List<string>();

    public string FailureNotificationsQueue { get; set; } = string.Empty;
    public string SuccessNotificationsQueue { get; set; } = string.Empty;
}

public static void IngestSingleFile(string file, string db, string table, string ingestionMappingRef)
{
    // Your Azure Data Explorer ingestion service URI, typically ingest-<your cluster name>.kusto.chinacloudapi.cn
    string DmServiceBaseUri = @"https://ingest-{serviceNameAndRegion}.kusto.chinacloudapi.cn";

    // 1. Authenticate the interactive user (or application) to access Kusto ingestion service
    string bearerToken = AuthenticateInteractiveUser(DmServiceBaseUri);

    // 2a. Retrieve ingestion resources
    IngestionResourcesSnapshot ingestionResources = RetrieveIngestionResources(DmServiceBaseUri, bearerToken);

    // 2b. Retrieve Kusto identity token
    string identityToken = RetrieveKustoIdentityToken(DmServiceBaseUri, bearerToken);

    // 3. Upload file to one of the blob containers we got from Azure Data Explorer.
    // This example uses the first one, but when working with multiple blobs,
    // one should round-robin the containers in order to prevent throttling
    long blobSizeBytes = 0;
    string blobName = $"TestData{DateTime.UtcNow.ToString("yyyy-MM-dd_HH-mm-ss.FFF")}";
    string blobUriWithSas = UploadFileToBlobContainer(file, ingestionResources.TempStorageContainers.First(),
                                                            "temp001", blobName, out blobSizeBytes);

    // 4. Compose ingestion command
    string ingestionMessage = PrepareIngestionMessage(db, table, blobUriWithSas, blobSizeBytes, ingestionMappingRef, identityToken);

    // 5. Post ingestion command to one of the ingestion queues we got from Azure Data Explorer.
    // This example uses the first one, but when working with multiple blobs,
    // one should round-robin the queues in order to prevent throttling
    PostMessageToQueue(ingestionResources.IngestionQueues.First(), ingestionMessage);

    Thread.Sleep(20000);

    // 6a. Read success notifications
    var successes = PopTopMessagesFromQueue(ingestionResources.SuccessNotificationsQueue, 32);
    foreach (var sm in successes)
    {
        Console.WriteLine($"Ingestion completed: {sm}");
    }

    // 6b. Read failure notifications
    var errors = PopTopMessagesFromQueue(ingestionResources.FailureNotificationsQueue, 32);
    foreach (var em in errors)
    {
        Console.WriteLine($"Ingestion error: {em}");
    }
}
```

## <a name="using-queued-ingestion-to-azure-data-explorer-for-production-grade-pipelines"></a>使用目标为 Azure 数据资源管理器的针对生产级管道的排队引入

### <a name="obtain-authentication-evidence-from-azure-ad"></a>从 Azure AD 获取身份验证证据

在这里，我们使用 ADAL 来获取 Azure AD 令牌以访问 Kusto 数据管理服务并请求其输入队列。
如果需要，可以在[非 Windows 平台](https://docs.azure.cn/active-directory/develop/active-directory-authentication-libraries)上使用 ADAL。

```csharp
// Authenticates the interactive user and retrieves Azure AD Access token for specified resource
internal static string AuthenticateInteractiveUser(string resource)
{
    // Create Auth Context for MSFT Azure AD:
    AuthenticationContext authContext = new AuthenticationContext("https://login.partner.microsoftonline.cn/{Azure AD Tenant ID or name}");

    // Acquire user token for the interactive user for Azure Data Explorer:
    AuthenticationResult result =
        authContext.AcquireTokenAsync(resource, "<your client app ID>", new Uri(@"<your client app URI>"),
                                        new PlatformParameters(PromptBehavior.Auto), UserIdentifier.AnyUser, "prompt=select_account").Result;
    return result.AccessToken;
}
```

### <a name="retrieve-azure-data-explorer-ingestion-resources"></a>检索 Azure 数据资源管理器引入资源

手动构造针对数据管理服务的 HTTP POST 请求，用以请求返回引入资源。 这些资源包括 DM 服务正在侦听的队列，以及用于数据上传的 blob 容器。
数据管理服务将处理其包含的引入请求已到达这些队列之一的任何消息。

```csharp
// Retrieve ingestion resources (queues and blob containers) with SAS from specified Azure Data Explorer Ingestion service using supplied Access token
internal static IngestionResourcesSnapshot RetrieveIngestionResources(string ingestClusterBaseUri, string accessToken)
{
    string ingestClusterUri = $"{ingestClusterBaseUri}/v1/rest/mgmt";
    string requestBody = $"{{ \"csl\": \".get ingestion resources\" }}";

    IngestionResourcesSnapshot ingestionResources = new IngestionResourcesSnapshot();

    using (WebResponse response = SendPostRequest(ingestClusterUri, accessToken, requestBody))
    using (StreamReader sr = new StreamReader(response.GetResponseStream()))
    using (JsonTextReader jtr = new JsonTextReader(sr))
    {
        JObject responseJson = JObject.Load(jtr);
        IEnumerable<JToken> tokens;

        // Input queues
        tokens = responseJson.SelectTokens("Tables[0].Rows[?(@.[0] == 'SecuredReadyForAggregationQueue')]");
        foreach (var token in tokens)
        {
            ingestionResources.IngestionQueues.Add((string) token[1]);
        }

        // Temp storage containers
        tokens = responseJson.SelectTokens("Tables[0].Rows[?(@.[0] == 'TempStorage')]");
        foreach (var token in tokens)
        {
            ingestionResources.TempStorageContainers.Add((string)token[1]);
        }

        // Failure notifications queue
        var singleToken =
            responseJson.SelectTokens("Tables[0].Rows[?(@.[0] == 'FailedIngestionsQueue')].[1]").FirstOrDefault();
        ingestionResources.FailureNotificationsQueue = (string)singleToken;

        // Success notifications queue
        singleToken =
            responseJson.SelectTokens("Tables[0].Rows[?(@.[0] == 'SuccessfulIngestionsQueue')].[1]").FirstOrDefault();
        ingestionResources.SuccessNotificationsQueue = (string)singleToken;
    }

    return ingestionResources;
}

// Executes a POST request on provided URI using supplied Access token and request body
internal static WebResponse SendPostRequest(string uriString, string authToken, string body)
{
    WebRequest request = WebRequest.Create(uriString);

    request.Method = "POST";
    request.ContentType = "application/json";
    request.ContentLength = body.Length;
    request.Headers.Set(HttpRequestHeader.Authorization, $"Bearer {authToken}");

    Stream bodyStream = request.GetRequestStream();
    using (StreamWriter sw = new StreamWriter(bodyStream))
    {
        sw.Write(body);
        sw.Flush();
    }

    bodyStream.Close();
    return request.GetResponse();
}
```

### <a name="obtain-a-kusto-identity-token"></a>获取 Kusto 标识令牌

引入消息通过非直接通道（Azure 队列）交付到 Azure 数据资源管理器，因此无法进行带内授权验证以访问 Azure 数据资源管理器引入服务。 解决方案是向每个引入消息附加一个标识令牌。 此令牌将启用带内授权验证。 然后，Azure 数据资源管理器服务就可以在收到引入消息时验证这个已签名的令牌。

```csharp
// Retrieves a Kusto identity token that will be added to every ingest message
internal static string RetrieveKustoIdentityToken(string ingestClusterBaseUri, string accessToken)
{
    string ingestClusterUri = $"{ingestClusterBaseUri}/v1/rest/mgmt";
    string requestBody = $"{{ \"csl\": \".get kusto identity token\" }}";
    string jsonPath = "Tables[0].Rows[*].[0]";

    using (WebResponse response = SendPostRequest(ingestClusterUri, accessToken, requestBody))
    using (StreamReader sr = new StreamReader(response.GetResponseStream()))
    using (JsonTextReader jtr = new JsonTextReader(sr))
    {
        JObject responseJson = JObject.Load(jtr);
        JToken identityToken = responseJson.SelectTokens(jsonPath).FirstOrDefault();

        return ((string)identityToken);
    }
}
```

### <a name="upload-data-to-the-azure-blob-container"></a>将数据上传到 Azure Blob 容器

此步骤介绍了如何将本地文件上传到 Azure Blob，以便交付该文件进行引入。 此代码使用了 Azure 存储 SDK。 如果无法使用依赖项，则可以通过 [Azure Blob 服务 REST API](https://docs.microsoft.com/rest/api/storageservices/fileservices/blob-service-rest-api) 实现此目的。

```csharp
// Uploads a single local file to an Azure Blob container, returns blob URI and original data size
internal static string UploadFileToBlobContainer(string filePath, string blobContainerUri, string containerName, string blobName, out long blobSize)
{
    var blobUri = new Uri(blobContainerUri);
    CloudBlobContainer blobContainer = new CloudBlobContainer(blobUri);
    CloudBlockBlob blockBlob = blobContainer.GetBlockBlobReference(blobName);

    using (Stream stream = File.OpenRead(filePath))
    {
        blockBlob.UploadFromStream(stream);
        blobSize = blockBlob.Properties.Length;
    }

    return string.Format("{0}{1}", blockBlob.Uri.AbsoluteUri, blobUri.Query);
}
```

### <a name="compose-the-azure-data-explorer-ingestion-message"></a>撰写 Azure 数据资源管理器引入消息

NewtonSoft.JSON 包将再次撰写有效的引入请求，该请求用于标识目标数据库和表并指向 blob。
该消息将发布到相关 Kusto 数据管理服务正在侦听的 Azure 队列。

下面是需要注意的一些要点。

* 此请求是引入消息的最小量。

> [!NOTE]
> 标识令牌是必需的，并且必须是 **AdditionalProperties** JSON 对象的一部分。

* 必要时，还必须提供 CsvMapping 或 JsonMapping 属性
* 有关详细信息，请参阅[有关预先创建引入映射的文章](../../management/create-ingestion-mapping-command.md)。
* [引入消息内部结构](#ingestion-message-internal-structure)部分解释了引入消息结构

```csharp
internal static string PrepareIngestionMessage(string db, string table, string dataUri, long blobSizeBytes, string mappingRef, string identityToken)
{
    var message = new JObject();

    message.Add("Id", Guid.NewGuid().ToString());
    message.Add("BlobPath", dataUri);
    message.Add("RawDataSize", blobSizeBytes);
    message.Add("DatabaseName", db);
    message.Add("TableName", table);
    message.Add("RetainBlobOnSuccess", true);   // Do not delete the blob on success
    message.Add("FlushImmediately", true);      // Do not aggregate
    message.Add("ReportLevel", 2);              // Report failures and successes (might incur perf overhead)
    message.Add("ReportMethod", 0);             // Failures are reported to an Azure Queue

    message.Add("AdditionalProperties", new JObject(
                                            new JProperty("authorizationContext", identityToken),
                                            new JProperty("jsonMappingReference", mappingRef),
                                            // Data is in JSON format
                                            new JProperty("format", "json")));
    return message.ToString();
}
```

### <a name="post-the-azure-data-explorer-ingestion-message-to-the-azure-data-explorer-ingestion-queue"></a>将 Azure 数据资源管理器引入消息发布到 Azure 数据资源管理器引入队列

最后，将你构造的消息发布到你从 Azure 数据资源管理器获取的选定引入队列。

> [!NOTE]
> 默认情况下，低于 v12 的 .Net 存储客户端版本将消息编码为 base64。有关详细信息，请参阅[存储文档](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.queue.cloudqueue.encodemessage?view=azure-dotnet-legacy#Microsoft_WindowsAzure_Storage_Queue_CloudQueue_EncodeMessage)。如果使用的是 v12 以上的 .Net 存储客户端版本，则必须正确编码消息内容。

```csharp
internal static void PostMessageToQueue(string queueUriWithSas, string message)
{
    CloudQueue queue = new CloudQueue(new Uri(queueUriWithSas));
    CloudQueueMessage queueMessage = new CloudQueueMessage(message);

    queue.AddMessage(queueMessage, null, null, null, null);
}
```

### <a name="check-for-error-messages-from-the-azure-queue"></a>检查是否有来自 Azure 队列的错误消息

在引入后，我们将检查数据管理向其中写入数据的相关队列是否返回了失败消息。 有关失败消息结构的详细信息，请参阅[引入失败消息结构](#ingestion-failure-message-structure)。 

```csharp
internal static IEnumerable<string> PopTopMessagesFromQueue(string queueUriWithSas, int count)
{
    List<string> messages = Enumerable.Empty<string>().ToList();
    CloudQueue queue = new CloudQueue(new Uri(queueUriWithSas));
    var messagesFromQueue = queue.GetMessages(count);
    foreach (var m in messagesFromQueue)
    {
        messages.Add(m.AsString);
        queue.DeleteMessage(m);
    }

    return messages;
}
```

## <a name="ingestion-messages---json-document-formats"></a>引入消息 - JSON 文档格式

### <a name="ingestion-message-internal-structure"></a>引入消息内部结构

Kusto 数据管理服务预期从输入 Azure 队列中读取的消息是采用以下格式的 JSON 文档。

```JSON
{
    "Id" : "<Id>",
    "BlobPath" : "https://<AccountName>.blob.core.chinacloudapi.cn/<ContainerName>/<PathToBlob>?<SasToken>",
    "RawDataSize" : "<RawDataSizeInBytes>",
    "DatabaseName": "<DatabaseName>",
    "TableName" : "<TableName>",
    "RetainBlobOnSuccess" : "<RetainBlobOnSuccess>",
    "FlushImmediately": "<true|false>",
    "ReportLevel" : <0-Failures, 1-None, 2-All>,
    "ReportMethod" : <0-Queue, 1-Table>,
    "AdditionalProperties" : { "<PropertyName>" : "<PropertyValue>" }
}
```

|属性 | 说明 |
|---------|-------------|
|ID |消息标识符 (GUID) |
|BlobPath |Blob 的路径 (URI)，包括向 Azure 数据资源管理器授予读取/写入/删除它的权限的 SAS 密钥。 Azure 数据资源管理器必须具有相关权限才能在完成数据引入后删除 blob|
|RawDataSize |解压缩的数据的大小（字节）。 提供此值将使得 Azure 数据资源管理器能够通过聚合多个 blob（在可能情况下）来优化引入。 此属性是可选的，但如果未提供，则 Azure 数据资源管理器将访问 blob 来检索该大小 |
|DatabaseName |目标数据库名称 |
|TableName |目标表名称 |
|RetainBlobOnSuccess |如果设置为 `true`，则成功完成引入后，不会删除 blob。 默认为 `false` |
|FlushImmediately |如果设置为 `true`，则会跳过任何聚合。 默认为 `false` |
|ReportLevel |成功/错误报告级别：0 - 失败、1 - 无、2 - 全部 |
|ReportMethod |报告机制：0 - 队列、1 - 表 |
|AdditionalProperties |其他属性，例如 `format`、`tags` 和 `creationTime`。 有关详细信息，请参阅[数据引入属性](../../../ingestion-properties.md)。|

### <a name="ingestion-failure-message-structure"></a>引入失败消息结构

数据管理预期从输入 Azure 队列中读取的消息需要是采用以下格式的 JSON 文档。

|属性 | 说明 |
|---------|-------------
|OperationId |可用于在服务端跟踪操作的操作标识符 (GUID) |
|数据库 |目标数据库名称 |
|表 |目标表名称 |
|FailedOn |失败时间戳 |
|IngestionSourceId |GUID，用于标识 Azure 数据资源管理器未能引入的数据块 |
|IngestionSourcePath |Azure 数据资源管理器未能引入的数据块的路径 (URI) |
|详细信息 |失败消息 |
|ErrorCode |Azure 数据资源管理器错误代码（请在[此处](kusto-ingest-client-errors.md#ingestion-error-codes)查看所有错误代码） |
|FailureStatus |指示失败是永久性的还是暂时性的 |
|RootActivityId |可用于在服务端跟踪操作的 Azure 数据资源管理器相关标识符 (GUID) |
|OriginatesFromUpdatePolicy |指示失败是否由错误的[事务性更新策略](../../management/updatepolicy.md)导致 |
|ShouldRetry | 指示如果按原样重试引入是否可以成功 |
