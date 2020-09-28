---
title: 使用 Azure 媒体服务 v3 指示描述性音频轨道 | Microsoft Docs
description: 按照本教程中的步骤，使用媒体服务 v3 上传文件、编码视频、添加描述性音频轨道和流式传输内容。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.custom: devx-track-csharp
origin.date: 08/31/220
ms.date: 09/28/2020
ms.author: v-jay
ms.openlocfilehash: e37ade2763b92ede7176d33907473a17c965a5b5
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91244959"
---
# <a name="signal-descriptive-audio-tracks"></a>指示描述性音频轨道

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

可以向视频中添加旁白轨道，以帮助视力受损的客户通过倾听旁白来跟随视频录制。 在媒体服务 v3 中，我们通过在清单文件中注释音频轨道来指示描述性音频轨道。

本文介绍如何编码视频、如何将包含描述性音频的仅限音频的 MP4 文件（AAC 编解码器）上传到输出资产中，以及如何编辑 .ism 文件，使之包含描述性音频。

## <a name="prerequisites"></a>先决条件

- [创建媒体服务帐户](./create-account-howto.md)。
- 遵循[使用 Azure CLI 访问 Azure 媒体服务 API](./access-api-howto.md) 中的步骤并保存凭据。 需要使用这些凭据来访问 API。
- 参阅[动态打包](dynamic-packaging-overview.md)。
- 参阅[上传、编码和流式传输视频](stream-files-tutorial-with-api.md)教程。

## <a name="create-an-input-asset-and-upload-a-local-file-into-it"></a>创建输入资产并将本地文件上传到该资产 

CreateInputAsset 函数创建新的输入[资产](https://docs.microsoft.com/rest/api/media/assets)并将指定的本地视频文件上传到该资产  。 此**资产**用作编码作业的输入。 在媒体服务 v3 中，**作业**输入可以是**资产**，也可以是可通过 HTTPS URL 使用媒体服务帐户访问的内容。 

如果想要了解如何从 HTTPS URL 进行编码，请参阅[此文](job-input-from-http-how-to.md)。  

在媒体服务 v3 中，使用 Azure 存储 API 上传文件。 以下 .NET 片段显示如何上传。

以下函数执行以下操作：

* 创建**资产** 
* 获取资产的[存储中容器](../../storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-a-container)的可写 [SAS URL](../../storage/common/storage-sas-overview.md)
* 使用 SAS URL 将文件上传到存储中的容器中

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

如果需要将已创建的输入资产的名称传递给其他方法，请务必使用从 `CreateInputAssetAsync` 返回的资产对象上的 `Name` 属性，例如 inputAsset.Name。 

## <a name="create-an-output-asset-to-store-the-result-of-the-encoding-job"></a>创建一个输出资产以存储编码作业的结果

输出[资产](https://docs.microsoft.com/rest/api/media/assets)会存储作业编码的结果。 以下函数说明了如何创建输出资产。

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
        string uniqueness = $"-{Guid.NewGuid().ToString("N")}";
        outputAssetName += uniqueness;
        
        Console.WriteLine("Warning – found an existing Asset with name = " + assetName);
        Console.WriteLine("Creating an Asset with this name instead: " + outputAssetName);                
    }

    return await client.Assets.CreateOrUpdateAsync(resourceGroupName, accountName, outputAssetName, asset);
}
```

如果需要将已创建的输出资产的名称传递给其他方法，请务必使用从 `CreateIOutputAssetAsync` 返回的资产对象上的 `Name` 属性，例如 outputAsset.Name。 

在本文中，请将 `outputAsset.Name` 值传递给 `SubmitJobAsync` 和 `UploadAudioIntoOutputAsset` 函数。

## <a name="create-a-transform-and-a-job-that-encodes-the-uploaded-file"></a>创建转换和一个对上传的文件进行编码的作业

对媒体服务中的内容进行编码或处理时，一种常见的模式是将编码设置设为脚本。 然后，需提交**作业**，将该脚本应用于视频。 为每个新视频提交新作业后，可将该脚本应用到库中的所有视频。 媒体服务中的脚本称为**转换**。 有关详细信息，请参阅[转换和作业](./transforms-jobs-concept.md)。 本教程中的示例定义有关将视频进行编码以将其流式传输到各种 iOS 和 Android 设备的脚本。 

以下示例创建一个转换（如果不存在）。

```csharp
private static async Task<Transform> GetOrCreateTransformAsync(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string transformName)
{
    // Does a Transform already exist with the desired name? Assume that an existing Transform with the desired name
    // also uses the same recipe or Preset for processing content.
    Transform transform = await client.Transforms.GetAsync(resourceGroupName, accountName, transformName);

    if (transform == null)
    {
        // You need to specify what you want it to produce as an output
        TransformOutput[] output = new TransformOutput[]
        {
            new TransformOutput
            {
                // The preset for the Transform is set to one of Media Services built-in sample presets.
                // You can  customize the encoding settings by changing this to use "StandardEncoderPreset" class.
                Preset = new BuiltInStandardEncoderPreset()
                {
                    // This sample uses the built-in encoding preset for Adaptive Bitrate Streaming.
                    PresetName = EncoderNamedPreset.AdaptiveStreaming
                }
            }
        };

        // Create the Transform with the output defined above
        transform = await client.Transforms.CreateOrUpdateAsync(resourceGroupName, accountName, transformName, output);
    }

    return transform;
}
```

以下函数提交了一个作业。

```csharp
private static async Task<Job> SubmitJobAsync(IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string transformName,
    string jobName,
    string inputAssetName,
    string outputAssetName)
{
    // Use the name of the created input asset to create the job input.
    JobInput jobInput = new JobInputAsset(assetName: inputAssetName);

    JobOutput[] jobOutputs =
    {
        new JobOutputAsset(outputAssetName),
    };

    // In this example, we are assuming that the job name is unique.
    //
    // If you already have a job with the desired name, use the Jobs.Get method
    // to get the existing job. In Media Services v3, the Get method on entities returns null 
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

## <a name="wait-for-the-job-to-complete"></a>等待作业完成

此作业需要一些时间才能完成，完成时可发出通知。 建议使用事件网格等待作业完成。

作业通常会经历以下状态：**已计划**、**已排队**、**正在处理**、**已完成**（最终状态）。 如果作业出错，则显示“错误”状态  。 如果作业正处于取消过程中，则显示“正在取消”，完成时则显示“已取消”   。

有关详细信息，请参阅[处理事件网格事件](reacting-to-media-services-events.md)。

## <a name="upload-the-audio-only-mp4-file"></a>上传仅含音频的 MP4 文件

将包含描述性音频的其他仅限音频的 MP4 文件（AAC 编解码器）上传到输出资产中。  

```csharp
private static async Task UpoadAudioIntoOutputAsset(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string outputAssetName,
    string fileToUpload)
{
    // Use the Assets.Get method to get the existing asset. 
    // In Media Services v3, the Get method on entities returns null 
    // if the entity doesn't exist (a case-insensitive check on the name).

    // Call Media Services API to create an Asset.
    // This method creates a container in storage for the Asset.
    // The files (blobs) associated with the asset will be stored in this container.
    Asset asset = await client.Assets.GetAsync(resourceGroupName, accountName, outputAssetName);
    
    if (asset != null)
    {
      // Use Media Services API to get back a response that contains
      // SAS URL for the Asset container into which to upload blobs.
      // That is where you would specify read-write permissions 
      // and the exparation time for the SAS URL.
      var response = await client.Assets.ListContainerSasAsync(
          resourceGroupName,
          accountName,
          outputAssetName,
          permissions: AssetContainerPermission.ReadWrite,
          expiryTime: DateTime.UtcNow.AddHours(4).ToUniversalTime());

      var sasUri = new Uri(response.AssetContainerSasUrls.First());

      // Use Storage API to get a reference to the Asset container
      // that was created by calling Asset's CreateOrUpdate method.  
      CloudBlobContainer container = new CloudBlobContainer(sasUri);
      var blob = container.GetBlockBlobReference(Path.GetFileName(fileToUpload));

      // Use Strorage API to upload the file into the container in storage.
      await blob.UploadFromFileAsync(fileToUpload);
    }
}
```

下面是一个调用 `UpoadAudioIntoOutputAsset` 函数的示例：

```csharp
await UpoadAudioIntoOutputAsset(client, config.ResourceGroup, config.AccountName, outputAsset.Name, "audio_description.m4a");
```

## <a name="edit-the-ism-file"></a>编辑 .ism 文件

当编码作业完成后，输出资产会包含编码作业生成的文件。 

1. 在 Azure 门户中，导航到与媒体服务帐户关联的存储帐户。 
1. 找到其名称为你的输出资产的容器。 
1. 在容器中找到 .ism 文件，然后单击“编辑 Blob”（在右窗口中）。 
1. 通过添加已上传的包含描述性音频的仅限音频的 MP4 文件（AAC 编解码器）的相关信息，对 .ism 文件进行编辑，在完成后按“保存”。

    若要指示描述性音频轨道，需将“accessibility”和“role”参数添加到 .ism 文件。 你有责任正确设置这些参数，以将音频轨道作为音频描述发信号。 例如，将 `<param name="accessibility" value="description" />` 和 `<param name="role" value="alternate" />` 添加到特定音频轨道的 .ism 文件中，如以下示例所示。
 
```xml
<?xml version="1.0" encoding="utf-8"?>
<smil xmlns="http://www.w3.org/2001/SMIL20/Language">
  <head>
    <meta name="clientManifestRelativePath" content="ignite.ismc" />
    <meta name="formats" content="mp4-v3" />
  </head>
  <body>
    <switch>
      <audio src="ignite_320x180_AACAudio_381.mp4" systemBitrate="128041" systemLanguage="eng">
        <param name="systemBitrate" value="128041" valuetype="data" />
        <param name="trackID" value="2" valuetype="data" />
        <param name="trackName" value="aac_eng_2_128041_2_1" valuetype="data" />
        <param name="systemLanguage" value="eng" valuetype="data" />
        <param name="trackIndex" value="ignite_320x180_AACAudio_381_2.mpi" valuetype="data" />
      </audio>
      <audio src="audio_description.m4a" systemBitrate="194000" systemLanguage="eng">
        <param name="trackName" value="aac_eng_audio_description" />
        <param name="accessibility" value="description" />
        <param name="role" value="alternate" />     
      </audio>          
      <video src="ignite_1280x720_AACAudio_3549.mp4" systemBitrate="3549855">
        <param name="systemBitrate" value="3549855" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_1280x720_AACAudio_3549_1.mpi" valuetype="data" />
      </video>
      <video src="ignite_960x540_AACAudio_2216.mp4" systemBitrate="2216764">
        <param name="systemBitrate" value="2216764" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_960x540_AACAudio_2216_1.mpi" valuetype="data" />
      </video>
      <video src="ignite_640x360_AACAudio_1154.mp4" systemBitrate="1154569">
        <param name="systemBitrate" value="1154569" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_640x360_AACAudio_1154_1.mpi" valuetype="data" />
      </video>
      <video src="ignite_480x270_AACAudio_721.mp4" systemBitrate="721893">
        <param name="systemBitrate" value="721893" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_480x270_AACAudio_721_1.mpi" valuetype="data" />
      </video>
      <video src="ignite_320x180_AACAudio_381.mp4" systemBitrate="381027">
        <param name="systemBitrate" value="381027" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_320x180_AACAudio_381_1.mpi" valuetype="data" />
      </video>
    </switch>
  </body>
</smil>
```

## <a name="get-a-streaming-locator"></a>获取流定位符

编码完成后，下一步是使输出资产中的视频可供客户端播放。 可通过两个步骤完成此操作：首先创建[流定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)，然后生成客户端可以使用的流 URL。 

创建**流定位符**的过程称为发布。 默认情况下，除非配置可选的开始和结束时间，否则调用 API 后，**流定位符**立即生效，并持续到被删除为止。 

创建 [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators) 时，需要指定所需的 StreamingPolicyName。 在此示例中将流式传输明文（或未加密的内容），因此使用预定义的明文流式传输策略 (**PredefinedStreamingPolicy.ClearStreamingOnly**)。

> [!IMPORTANT]
> 使用自定义的[流策略](https://docs.microsoft.com/rest/api/media/streamingpolicies)时，应为媒体服务帐户设计有限的一组此类策略，并在需要同样的加密选项和协议时重新将这些策略用于 StreamingLocators。 媒体服务帐户具有对应于流式处理策略条目数的配额。 不应为每个流式处理定位符创建新的流式处理策略。

以下代码假定使用唯一的 locatorName 调用该函数。

```csharp
private static async Task<StreamingLocator> CreateStreamingLocatorAsync(
    IAzureMediaServicesClient client,
    string resourceGroup,
    string accountName,
    string assetName,
    string locatorName)
{
    StreamingLocator locator = await client.StreamingLocators.CreateAsync(
        resourceGroup,
        accountName,
        locatorName,
        new StreamingLocator
        {
            AssetName = assetName,
            StreamingPolicyName = PredefinedStreamingPolicy.ClearStreamingOnly
        });

    return locator;
}
```

虽然本主题的示例讨论的是流式传输，但可以使用相同的调用创建一个流定位符，通过渐进式下载的方式来提供视频。

### <a name="get-streaming-urls"></a>获取流式 URL

创建[流定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)后，可以获取流 URL，如 **GetStreamingURLs** 中所示。 若要生成 URL，需要连接[流式处理终结点](https://docs.microsoft.com/rest/api/media/streamingendpoints)的主机名和**流定位符**路径。 此示例使用默认的**流式处理终结点**。 首次创建媒体服务帐户时，此默认的**流式处理终结点**处于停止状态，因此需要调用 **Start**。

> [!NOTE]
> 在此方法中，需要指定在创建输出资产的**流定位符**时所用的 locatorName。

```csharp
private static async Task<IList<string>> GetStreamingUrlsAsync(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    String locatorName)
{
    const string DefaultStreamingEndpointName = "default";

    IList<string> streamingUrls = new List<string>();

    StreamingEndpoint streamingEndpoint = await client.StreamingEndpoints.GetAsync(resourceGroupName, accountName, DefaultStreamingEndpointName);

    if (streamingEndpoint != null)
    {
        if (streamingEndpoint.ResourceState != StreamingEndpointResourceState.Running)
        {
            await client.StreamingEndpoints.StartAsync(resourceGroupName, accountName, DefaultStreamingEndpointName);
        }
    }

    ListPathsResponse paths = await client.StreamingLocators.ListPathsAsync(resourceGroupName, accountName, locatorName);

    foreach (StreamingPath path in paths.StreamingPaths)
    {
        UriBuilder uriBuilder = new UriBuilder();
        uriBuilder.Scheme = "https";
        uriBuilder.Host = streamingEndpoint.HostName;

        uriBuilder.Path = path.Paths[0];
        streamingUrls.Add(uriBuilder.ToString());
    }

    return streamingUrls;
}
```

## <a name="test-with-azure-media-player"></a>使用 Azure Media Player 进行测试

本文使用 Azure Media Player 测试流式传输。 

> [!NOTE]
> 如果播放器在 Https 站点上进行托管，请确保将 URL 更新为“https”。

1. 打开 Web 浏览器并导航到 [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/)。
2. 在“URL:”框中，粘贴从应用程序获取的某个流式处理 URL 值。 
 
     可以粘贴 HLS、Dash 或 Smooth 格式的 URL，Azure Media Player将切换到适当的流协议，以便在你的设备上自动播放。
3. 按“更新播放器”。

Azure Media Player 可用于测试，但不可在生产环境中使用。 

