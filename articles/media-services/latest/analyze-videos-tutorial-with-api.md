---
title: 使用媒体服务 v3 来分析视频
titleSuffix: Azure Media Services
description: 了解如何使用 Azure 媒体服务分析视频。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: tutorial
origin.date: 08/31/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.custom: seodec18
ms.openlocfilehash: 8e43eb0dd32e0858ae8baa4b9652e5de2105b5d1
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245741"
---
# <a name="tutorial-analyze-videos-with-media-services-v3"></a>教程：使用媒体服务 v3 来分析视频

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

> [!NOTE]
> 尽管本教程使用了 [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.models.liveevent?view=azure-dotnet) 示例，但 [REST API](https://docs.microsoft.com/rest/api/media/liveevents)、[CLI](https://docs.microsoft.com/cli/azure/ams/live-event?view=azure-cli-latest) 或其他受支持的 [SDK](media-services-apis-overview.md#sdks) 的常规步骤是相同的。

本教程介绍如何使用 Azure 媒体服务分析视频。 在很多情况下，用户可能会希望深入了解录制的视频或音频内容。 例如，若要提高客户满意度，组织可运行语音转文本处理，将客户支持录音转换为具有索引和仪表板的可搜索目录。 然后即可获得有关其业务的见解。 这些见解包括常见投诉列表、此类投诉的来源，以及其他有用信息。

本教程演示如何：

> [!div class="checklist"]
> * 下载本主题中所述的示例应用。
> * 检查用于分析指定视频的代码。
> * 运行应用。
> * 检查输出。
> * 清理资源。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="compliance-privacy-and-security"></a>符合性、隐私和安全性
 
需要重点提醒的是，在使用视频索引器时，你必须遵守所有适用法律，不得以侵犯他人权利或可能对他人有害的方式使用视频索引器或任何其他 Azure 服务。 在将任何视频（包括任何生物特征数据）上传到视频索引器服务进行处理和存储之前，必须拥有所有适当的权利，包括获得视频中个人的所有适当同意。 有关 Azure 的隐私义务和数据处理，请查看 Azure 的[隐私声明](https://www.azure.cn/support/legal/privacy-statement/)。 

## <a name="prerequisites"></a>先决条件

- 如果没有安装 Visual Studio，请下载 [Visual Studio Community 2019](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15)。
- [创建媒体服务帐户](./create-account-howto.md)。<br/>请务必记住用于资源组名称和媒体服务帐户名称的值。
- 遵循[使用 Azure CLI 访问 Azure 媒体服务 API](./access-api-howto.md) 中的步骤并保存凭据。 你将需要使用这些凭据来访问 API。

## <a name="download-and-configure-the-sample"></a>下载并配置示例

使用以下命令将包含 .NET 示例的 GitHub 存储库克隆到计算机：  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```

该示例位于 [AnalyzeVideos](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/AnalyzeVideos) 文件夹。

打开下载的项目中的 [appsettings.json](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/AnalyzeVideos/appsettings.json)。 将值替换为从[访问 API](./access-api-howto.md) 获得的凭据。

## <a name="examine-the-code-that-analyzes-the-specified-video"></a>检查用于分析指定视频的代码

本节讨论 AnalyzeVideos 项目的 [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/AnalyzeVideos/Program.cs) 文件中定义的函数。

该示例执行以下操作：

1. 创建**转换**和用于分析视频的**作业**。
2. 创建输入**资产**，并将视频上传到其中。 该资产用作作业的输入。
3. 创建用于存储作业输出的输出资产。
4. 提交作业。
5. 检查作业的状态。
6. 下载运行作业产生的文件。

> [!NOTE]
> 使用视频或音频分析器预设时，请通过 Azure 门户将帐户设置为具有 10 个 S3 媒体预留单位。 有关详细信息，请参阅[缩放媒体处理](media-reserved-units-cli-how-to.md)。

### <a name="start-using-media-services-apis-with-net-sdk"></a>开始结合使用媒体服务 API 与 .NET SDK

若要开始将媒体服务 API 与 .NET 结合使用，需要创建 AzureMediaServicesClient 对象。 若要创建对象，需要提供客户端所需凭据以使用 Azure AD 连接到 Azure。 在本文开头克隆的代码中，**GetCredentialsAsync** 函数根据本地配置文件中提供的凭据创建 ServiceClientCredentials 对象。 

```csharp
private static async Task<IAzureMediaServicesClient> CreateMediaServicesClientAsync(ConfigWrapper config)
{
    var credentials = await GetCredentialsAsync(config);

    return new AzureMediaServicesClient(config.ArmEndpoint, credentials)
    {
        SubscriptionId = config.SubscriptionId,
    };
}
```

### <a name="create-an-input-asset-and-upload-a-local-file-into-it"></a>创建输入资产并将本地文件上传到该资产 

CreateInputAsset 函数创建新的输入[资产](https://docs.microsoft.com/rest/api/media/assets)并将指定的本地视频文件上传到该资产  。 此资产用作编码作业的输入。 在媒体服务 v3 中，作业输入可以是资产，也可以是可通过 HTTPS URL 使用媒体服务帐户访问的内容。 要了解如何从 HTTPS URL 进行编码，请参阅[此](job-input-from-http-how-to.md)文章。  

在媒体服务 v3 中，使用 Azure 存储 API 上传文件。 以下 .NET 片段显示如何上传。

以下函数执行以下操作：

* 创建 Asset。
* 获取 Asset 的[存储中容器](../../storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-a-container)的可写 [SAS URL](../../storage/common/storage-sas-overview.md)。

    如果使用资产的 [ListContainerSas](https://docs.microsoft.com/rest/api/media/assets/listcontainersas) 函数获取 SAS URL，请注意，该函数将返回多个 SAS URL，因为每个存储帐户有两个存储帐户密钥。 存储帐户有两个密钥，因为它支持存储帐户密钥无缝轮换（例如，使用一个密钥时更改另一个，然后开始使用新密钥并轮换其他密钥）。 第一个 SAS URL 表示存储 key1，第二个表示存储 key2。
* 使用 SAS URL 将文件上传到存储中的容器中。

```csharp
private static async Task<Asset> CreateInputAssetAsync(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string assetName,
    string fileToUpload)
{
    // In this example, we are assuming that the asset name is unique.
    //
    // If you already have an asset with the desired name, use the Assets.Get method
    // to get the existing asset. In Media Services v3, the Get method on entities returns null 
    // if the entity doesn't exist (a case-insensitive check on the name).

    // Call Media Services API to create an Asset.
    // This method creates a container in storage for the Asset.
    // The files (blobs) associated with the asset will be stored in this container.
    Asset asset = await client.Assets.CreateOrUpdateAsync(resourceGroupName, accountName, assetName, new Asset());

    // Use Media Services API to get back a response that contains
    // SAS URL for the Asset container into which to upload blobs.
    // That is where you would specify read-write permissions 
    // and the exparation time for the SAS URL.
    var response = await client.Assets.ListContainerSasAsync(
        resourceGroupName,
        accountName,
        assetName,
        permissions: AssetContainerPermission.ReadWrite,
        expiryTime: DateTime.UtcNow.AddHours(4).ToUniversalTime());

    var sasUri = new Uri(response.AssetContainerSasUrls.First());

    // Use Storage API to get a reference to the Asset container
    // that was created by calling Asset's CreateOrUpdate method.  
    CloudBlobContainer container = new CloudBlobContainer(sasUri);
    var blob = container.GetBlockBlobReference(Path.GetFileName(fileToUpload));

    // Use Strorage API to upload the file into the container in storage.
    await blob.UploadFromFileAsync(fileToUpload);

    return asset;
}
```

### <a name="create-an-output-asset-to-store-the-result-of-the-job"></a>创建一个输出资产以存储作业的结果

输出[资产](/rest/api/media/assets)会存储作业结果。 项目定义 DownloadResults 函数，该函数将结果从此输出资产中下载到**输出**文件夹中，便于用户查看获取的内容。

```csharp
private static async Task<Asset> CreateOutputAssetAsync(IAzureMediaServicesClient client, string resourceGroupName, string accountName, string assetName)
{
    // Check if an Asset already exists
    Asset outputAsset = await client.Assets.GetAsync(resourceGroupName, accountName, assetName);
    Asset asset = new Asset();
    string outputAssetName = assetName;

    if (outputAsset != null)
    {
        // Name collision! In order to get the sample to work, let's just go ahead and create a unique asset name
        // Note that the returned Asset can have a different name than the one specified as an input parameter.
        // You may want to update this part to throw an Exception instead, and handle name collisions differently.
        string uniqueness = $"-{Guid.NewGuid():N}";
        outputAssetName += uniqueness;

        Console.WriteLine("Warning – found an existing Asset with name = " + assetName);
        Console.WriteLine("Creating an Asset with this name instead: " + outputAssetName);
    }

    return await client.Assets.CreateOrUpdateAsync(resourceGroupName, accountName, outputAssetName, asset);
}
```

### <a name="create-a-transform-and-a-job-that-analyzes-videos"></a>创建转换和分析视频的作业

对媒体服务中的内容进行编码或处理时，一种常见的模式是将编码设置设为脚本。 然后，需提交**作业**，将该脚本应用于视频。 为每个新视频提交新 Job 后，可将该脚本应用到库中的所有视频。 媒体服务中的脚本称为“转换”。 有关详细信息，请参阅[转换和作业](./transforms-jobs-concept.md)。 本教程中所述的示例定义了分析指定视频的脚本。

#### <a name="transform"></a>转换

创建新[转换](https://docs.microsoft.com/rest/api/media/transforms)实例时，需要指定希望生成的输出内容。 TransformOutput 是必需参数。 每个 TransformOutput 包含一个预设   。 预设介绍视频和/或音频处理操作的各个步骤，这些操作可生成所需 TransformOutput 。 在此示例中，使用了 **VideoAnalyzerPreset** 预设，并且将语言 (“en-US”) 传递给了其构造函数 (`new VideoAnalyzerPreset("en-US")`)。 凭借此预设，可以从视频提取多个音频和视频见解。 如需从视频提取多个音频见解，可以使用**AudioAnalyzerPreset** 预设。

创建 Transform 时，首先检查是否其中一个已存在使用 Get 方法，如下面的代码中所示 。 在媒体服务 v3 中，如果实体不存在（对名称进行不区分大小写检查），实体上的 **Get** 方法将返回 **null**。

```csharp
private static async Task<Transform> GetOrCreateTransformAsync(IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string transformName,
    Preset preset)
{
    // Does a Transform already exist with the desired name? Assume that an existing Transform with the desired name
    // also uses the same recipe or Preset for processing content.
    Transform transform = await client.Transforms.GetAsync(resourceGroupName, accountName, transformName);

    if (transform == null)
    {
        // Start by defining the desired outputs.
        TransformOutput[] outputs = new TransformOutput[]
        {
            new TransformOutput(preset),
        };

        // Create the Transform with the output defined above
        transform = await client.Transforms.CreateOrUpdateAsync(resourceGroupName, accountName, transformName, outputs);
    }

    return transform;
}
```

#### <a name="job"></a>作业

如上所述，[转换](https://docs.microsoft.com/rest/api/media/transforms)对象为脚本，[作业则是](https://docs.microsoft.com/rest/api/media/jobs)对媒体服务的实际请求，请求将转换应用到给定输入视频或音频内容。 Job 指定输入视频位置和输出位置等信息。 可以使用以下项指定视频的位置：HTTPS URL、SAS URL 或媒体服务帐户中的资产。

在此示例中，作业输入是一个本地视频。  

```csharp
private static async Task<Job> SubmitJobAsync(IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string transformName,
    string jobName,
    JobInput jobInput,
    string outputAssetName)
{
    JobOutput[] jobOutputs =
    {
        new JobOutputAsset(outputAssetName),
    };

    // In this example, we are assuming that the job name is unique.
    //
    // If you already have a job with the desired name, use the Jobs.Get method
    // to get the existing job. In Media Services v3, Get methods on entities returns null 
    // if the entity doesn't exist (a case-insensitive check on the name).
    Job job = await client.Jobs.CreateAsync(
        resourceGroupName,
        accountName,
        transformName,
        jobName,
        new Job
        {
            Input = jobInput,
            Outputs = jobOutputs,
        });

    return job;
}
```

### <a name="wait-for-the-job-to-complete"></a>等待作业完成

该作业需要一些时间才能完成操作。 在该过程中，你应能够接收通知。 可通过不同选项获取有关[作业](https://docs.microsoft.com/rest/api/media/jobs)完成情况的通知。 最简单的选项（如下所示）是使用轮询。

对于生产应用程序，由于可能出现延迟，并不建议将轮询作为最佳做法。 如果在帐户上过度使用轮询，轮询会受到限制。 开发者应改用事件网格。

事件网格旨在实现高可用性、一致性能和动态缩放。 使用事件网格，应用可以侦听和响应来自几乎所有 Azure 服务和自定义源的事件。 处理基于 HTTP 的反应事件非常简单，这有助于通过对事件的智能筛选和路由生成高效的解决方案。 有关详细信息，请参阅[将事件路由到自定义 Web 终结点](job-state-events-cli-how-to.md)。

**作业**通常会经历以下状态：**已计划**、**已排队**、**正在处理**、**已完成**（最终状态）。 如果作业出错，则显示“错误”状态。 如果作业正处于取消过程中，则显示“正在取消”，完成时则显示“已取消” 。

```csharp
private static async Task<Job> WaitForJobToFinishAsync(IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string transformName,
    string jobName)
{
    const int SleepIntervalMs = 20 * 1000;

    Job job;
    do
    {
        job = await client.Jobs.GetAsync(resourceGroupName, accountName, transformName, jobName);

        Console.WriteLine($"Job is '{job.State}'.");
        for (int i = 0; i < job.Outputs.Count; i++)
        {
            JobOutput output = job.Outputs[i];
            Console.Write($"\tJobOutput[{i}] is '{output.State}'.");
            if (output.State == JobState.Processing)
            {
                Console.Write($"  Progress (%): '{output.Progress}'.");
            }

            Console.WriteLine();
        }

        if (job.State != JobState.Finished && job.State != JobState.Error && job.State != JobState.Canceled)
        {
            await Task.Delay(SleepIntervalMs);
        }
    }
    while (job.State != JobState.Finished && job.State != JobState.Error && job.State != JobState.Canceled);

    return job;
}
```

### <a name="job-error-codes"></a>作业错误代码

请参阅[错误代码](https://docs.microsoft.com/rest/api/media/jobs/get#joberrorcode)。

### <a name="download-the-result-of-the-job"></a>下载作业结果

以下函数将输出 [Asset](https://docs.microsoft.com/rest/api/media/assets) 的结果下载到“输出”文件夹中，以便检查作业结果。

```csharp
private static async Task DownloadOutputAssetAsync(
    IAzureMediaServicesClient client,
    string resourceGroup,
    string accountName,
    string assetName,
    string outputFolderName)
{
    if (!Directory.Exists(outputFolderName))
    {
        Directory.CreateDirectory(outputFolderName);
    }

    AssetContainerSas assetContainerSas = await client.Assets.ListContainerSasAsync(
        resourceGroup,
        accountName,
        assetName,
        permissions: AssetContainerPermission.Read,
        expiryTime: DateTime.UtcNow.AddHours(1).ToUniversalTime());

    Uri containerSasUrl = new Uri(assetContainerSas.AssetContainerSasUrls.FirstOrDefault());
    CloudBlobContainer container = new CloudBlobContainer(containerSasUrl);

    string directory = Path.Combine(outputFolderName, assetName);
    Directory.CreateDirectory(directory);

    Console.WriteLine($"Downloading output results to '{directory}'...");

    BlobContinuationToken continuationToken = null;
    IList<Task> downloadTasks = new List<Task>();

    do
    {
        // A non-negative integer value that indicates the maximum number of results to be returned at a time,
        // up to the per-operation limit of 5000. If this value is null, the maximum possible number of results
        // will be returned, up to 5000.
        int? ListBlobsSegmentMaxResult = null;

        BlobResultSegment segment = await container.ListBlobsSegmentedAsync(null, true, BlobListingDetails.None, ListBlobsSegmentMaxResult, continuationToken, null, null);

        foreach (IListBlobItem blobItem in segment.Results)
        {
            if (blobItem is CloudBlockBlob blob)
            {
                string path = Path.Combine(directory, blob.Name);

                downloadTasks.Add(blob.DownloadToFileAsync(path, FileMode.Create));
            }
        }

        continuationToken = segment.ContinuationToken;
    }
    while (continuationToken != null);

    await Task.WhenAll(downloadTasks);

    Console.WriteLine("Download complete.");
}
```

### <a name="clean-up-resource-in-your-media-services-account"></a>清理媒体服务帐户中的资源

一般来说，除了打算重用的对象之外，应该清除所有对象（通常，将重用 Transform 并保留 StreamingLocators）。 如果希望帐户在试验后保持干净状态，则删除不打算重复使用的资源。 例如，以下代码可删除作业和输出资产：

```csharp
private static async Task CleanUpAsync(
   IAzureMediaServicesClient client,
   string resourceGroupName,
   string accountName,
   string transformName,
   string jobName,
   List<string> assetNames,
   string contentKeyPolicyName = null
   )
{
    await client.Jobs.DeleteAsync(resourceGroupName, accountName, transformName, jobName);

    foreach (var assetName in assetNames)
    {
        await client.Assets.DeleteAsync(resourceGroupName, accountName, assetName);
    }

    if (contentKeyPolicyName != null)
    {
        client.ContentKeyPolicies.Delete(resourceGroupName, accountName, contentKeyPolicyName);
    }
}
```

## <a name="run-the-sample-app"></a>运行示例应用

按 Ctrl+F5 运行 AnalyzeVideos 应用。

运行该程序时，作业会生成其在视频中发现的每张人脸的缩略图。 它还会生成 insights.json 文件。

## <a name="examine-the-output"></a>检查输出

分析视频得到的输出文件称为 insights.json。 此文件包含有关视频的见解。 可以从[媒体智能](./analyzing-video-audio-files-concept.md)一文中获取对 json 文件中找到的元素的说明。

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组中的任何一个资源（包括为本教程创建的媒体服务和存储帐户），请删除之前创建的资源组。

执行以下 CLI 命令：

```azurecli
az group delete --name amsResourceGroup
```

## <a name="multithreading"></a>多线程处理

Azure 媒体服务 v3 SDK 不是线程安全的。 使用多线程应用时，应在每个线程上生成一个新的 AzureMediaServicesClient 对象。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [教程：上传、编码和流式处理文件](stream-files-tutorial-with-api.md)
