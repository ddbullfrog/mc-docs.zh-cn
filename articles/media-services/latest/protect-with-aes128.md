---
title: 使用 AES-128 加密视频
titleSuffix: Azure Media Services
description: 了解如何使用 AES 128 位加密来加密视频，以及如何在 Azure 媒体服务中使用密钥传送服务。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
origin.date: 08/31/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.openlocfilehash: c00225a7851d09b1e2634cca0b776c0b28e60d81
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91244978"
---
# <a name="tutorial-encrypt-video-with-aes-128-and-use-the-key-delivery-service"></a>教程：使用 AES-128 来加密视频以及使用密钥传送服务

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

> [!NOTE]
> Google Widevine 内容保护服务目前在 Azure 中国区域不可用。

> [!NOTE]
> 尽管本教程使用了 [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.models.liveevent?view=azure-dotnet) 示例，但 [REST API](https://docs.microsoft.com/rest/api/media/liveevents)、[CLI](https://docs.microsoft.com/cli/azure/ams/live-event?view=azure-cli-latest) 或其他受支持的 [SDK](media-services-apis-overview.md#sdks) 的常规步骤是相同的。

借助媒体服务，可以传送使用 AES 通过 128 位加密密钥加密的 HTTP Live Streaming (HLS)、MPEG-DASH 和平滑流。 媒体服务还提供密钥传送服务，将加密密钥传送给已授权的用户。 如果希望媒体服务动态加密视频，需要将加密密钥与流定位器相关联，并配置内容密钥策略。 当播放器请求流时，媒体服务将使用指定的密钥通过 AES-128 动态加密内容。 为解密流，播放器从密钥传送服务请求密钥。 为了确定是否已授权用户获取密钥，服务将评估你为密钥指定的内容密钥策略。

可以使用多个加密类型（AES-128、PlayReady、FairPlay）来加密每个资产。 请参阅[流式处理协议和加密类型](content-protection-overview.md#streaming-protocols-and-encryption-types)，以了解有效的组合方式。 另请参阅[如何使用 DRM 进行保护](protect-with-drm.md)。

本文中示例的输出包括 Azure Media Player 的 URL、清单 URL，以及播放内容所需的 AES 令牌。 该示例将 JSON Web 令牌 (JWT) 令牌的过期时间设置为 1 小时。 可以打开浏览器并粘贴生成的 URL 来启动 Azure Media Player 演示页，其中已经填充了该 URL 和令牌（采用以下格式：```https://ampdemo.azureedge.net/?url= {dash Manifest URL} &aes=true&aestoken=Bearer%3D{ JWT Token here}```）。

本教程介绍了如何：

> [!div class="checklist"]
> * 下载本文中介绍的 [EncryptWithAES](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES) 示例。
> * 开始结合使用媒体服务 API 与 .NET SDK。
> * 创建输出资产。
> * 创建编码转换。
> * 提交作业。
> * 等待作业完成。
> * 创建内容密钥策略
> * 配置使用 JWT 令牌限制所需的策略。
> * 创建流式处理定位符。
> * 配置流定位器，以便使用 AES (ClearKey) 加密视频。
> * 获取测试令牌。
> * 生成流 URL。
> * 清理资源。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

以下是完成本教程所需具备的条件。

* 查看[内容保护概述](content-protection-overview.md)一文。
* 安装 Visual Studio Code 或 Visual Studio。
* [创建媒体服务帐户](./create-account-howto.md)。
* 根据[访问 API](./access-api-howto.md) 中所述，获取使用媒体服务 API 时所需的凭据。

## <a name="download-code"></a>下载代码

使用以下命令，将包含本文中所述完整 .NET 示例的 GitHub 存储库克隆到计算机：

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```

“使用 AES-128 加密”示例位于 [EncryptWithAES](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES) 文件夹中。

> [!NOTE]
> 每次运行应用时，该示例就会创建唯一的资源。 通常，我们会重复使用现有的资源，例如转换和策略（如果现有资源具有所需的配置）。

## <a name="start-using-media-services-apis-with-net-sdk"></a>开始结合使用媒体服务 API 与 .NET SDK

若要开始将媒体服务 API 与 .NET 结合使用，请创建 AzureMediaServicesClient 对象****。 若要创建对象，需要提供客户端所需凭据以使用 Azure AD 连接到 Azure。 在本文开头克隆的代码中，**GetCredentialsAsync** 函数根据本地配置文件中提供的凭据创建 ServiceClientCredentials 对象。

```c#
private static async Task<IAzureMediaServicesClient> CreateMediaServicesClientAsync(ConfigWrapper config)
{
    var credentials = await GetCredentialsAsync(config);

    return new AzureMediaServicesClient(config.ArmEndpoint, credentials)
    {
        SubscriptionId = config.SubscriptionId,
    };
}
```

## <a name="create-an-output-asset"></a>创建输出资产  

输出[资产](https://docs.microsoft.com/rest/api/media/assets)会存储作业编码的结果。  

```c#
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
 
## <a name="get-or-create-an-encoding-transform"></a>获取或创建编码转换

创建新实例时，需要指定希望生成的输出内容[转换](https://docs.microsoft.com/rest/api/media/transforms)。 所需参数是 **TransformOutput** 对象，如以下代码所示。 每个 TransformOutput 包含一个预设   。 预设介绍了视频和/或音频处理操作的分步说明，这些操作将用于生成所需的 TransformOutput 。 本文中的示例使用名为 AdaptiveStreaming 的内置预设。 此预设将输入的视频编码为基于输入的分辨率和比特率自动生成的比特率阶梯（比特率 - 分辨率对），然后通过与每个比特率 - 分辨率对相对应的 H.264 视频和 AAC 音频生成 ISO MP4 文件。

在创建新的[转换](https://docs.microsoft.com/rest/api/media/transforms)之前，请先检查是否已存在使用 **Get** 方法的转换，如以下代码中所示。 在 Media Services v3**获取**实体上的方法返回**null**如果实体不存在 （不区分大小写的名称检查）。

```c#
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

## <a name="submit-job"></a>提交作业

如上所述，[转换](https://docs.microsoft.com/rest/api/media/transforms)对象为脚本，[作业则是](https://docs.microsoft.com/rest/api/media/jobs)对媒体服务的实际请求，请求将转换应用到给定输入视频或音频内容。 Job 指定输入视频位置和输出位置等信息。

本教程基于直接从 [HTTPs 源 URL](job-input-from-http-how-to.md) 引入的文件创建作业的输入。

```c#
private static async Task<Job> SubmitJobAsync(IAzureMediaServicesClient client,
    string resourceGroup,
    string accountName,
    string transformName,
    string outputAssetName,
    string jobName)
{
    // This example shows how to encode from any HTTPs source URL - a new feature of the v3 API.  
    // Change the URL to any accessible HTTPs URL or SAS URL from Azure.
    JobInputHttp jobInput =
        new JobInputHttp(files: new[] { "https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/Ignite-short.mp4" });

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
        resourceGroup,
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

该作业需要一些时间才能完成操作。 在该过程中，你应能够接收通知。 以下代码示例显示如何轮询服务以获取[作业](https://docs.microsoft.com/rest/api/media/jobs)状态。 对于生产应用程序，由于可能出现延迟，并不建议将轮询作为最佳做法。 如果在帐户上过度使用轮询，轮询会受到限制。 开发者应改用事件网格。 有关详细信息，请参阅[将事件路由到自定义 Web 终结点](job-state-events-cli-how-to.md)。

**作业**通常会经历以下状态：**已计划**、**已排队**、**正在处理**、**已完成**（最终状态）。 如果作业出错，则显示“错误”状态。 如果作业正处于取消过程中，则显示“正在取消”，完成时则显示“已取消” 。

```c#
private static async Task<Job> WaitForJobToFinishAsync(IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string transformName,
    string jobName)
{
    const int SleepIntervalMs = 60 * 1000;

    Job job = null;

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
                Console.Write($"  Progress: '{output.Progress}'.");
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

## <a name="create-a-content-key-policy"></a>创建内容密钥策略

内容密钥提供对资产的安全访问。 需要创建一个内容密钥策略，用于配置如何将内容密钥传送到终端客户端。 内容密钥与**流定位器**相关联。 媒体服务还提供密钥传送服务，将加密密钥传送给已授权的用户。

当播放器请求流时，媒体服务将使用指定的密钥通过 AES 加密来动态加密内容（本例中使用 AES 加密）。为解密流，播放器从密钥传送服务请求密钥。 为了确定是否已授权用户获取密钥，服务将评估你为密钥指定的内容密钥策略。

```c#
private static async Task<ContentKeyPolicy> GetOrCreateContentKeyPolicyAsync(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string contentKeyPolicyName)
{
    ContentKeyPolicy policy = await client.ContentKeyPolicies.GetAsync(resourceGroupName, accountName, contentKeyPolicyName);

    if (policy == null)
    {

        ContentKeyPolicySymmetricTokenKey primaryKey = new ContentKeyPolicySymmetricTokenKey(TokenSigningKey);
        List<ContentKeyPolicyRestrictionTokenKey> alternateKeys = null;
        List<ContentKeyPolicyTokenClaim> requiredClaims = new List<ContentKeyPolicyTokenClaim>()
        {
            ContentKeyPolicyTokenClaim.ContentKeyIdentifierClaim
        };

        List<ContentKeyPolicyOption> options = new List<ContentKeyPolicyOption>()
        {
            new ContentKeyPolicyOption(
                new ContentKeyPolicyClearKeyConfiguration(),
                new ContentKeyPolicyTokenRestriction(Issuer, Audience, primaryKey,
                    ContentKeyPolicyRestrictionTokenType.Jwt, alternateKeys, requiredClaims))
        };

        // Since we are randomly generating the signing key each time, make sure to create or update the policy each time.
        // Normally you would use a long lived key so you would just check for the policies existence with Get instead of
        // ensuring to create or update it each time.
        policy = await client.ContentKeyPolicies.CreateOrUpdateAsync(resourceGroupName, accountName, contentKeyPolicyName, options);

    }

    return policy;
}
```

## <a name="create-a-streaming-locator"></a>创建流定位符

完成编码并设置内容密钥策略后，下一步是使输出资产中的视频可供客户端播放。 可以通过两个步骤来提供视频：

1. 创建[流式处理定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)。
2. 生成客户端可以使用的流式处理 URL。

创建**流定位器**的过程称为发布。 默认情况下，除非配置可选的开始和结束时间，否则调用 API 后，流式处理定位符**** 立即生效， 并持续到被删除为止。

创建[流式处理定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)时，需要指定所需的 **StreamingPolicyName**。 本教程使用某个 PredefinedStreamingPolicies 来告知 Azure 媒体服务如何发布流式处理的内容。 此示例中应用了 AES 信封加密（此加密也称为 ClearKey 加密，因为密钥是通过 HTTPS 而不是 DRM 许可证传送到播放客户端的）。

> [!IMPORTANT]
> 使用自定义的 [StreamingPolicy](https://docs.microsoft.com/rest/api/media/streamingpolicies) 时，应为媒体服务帐户设计有限的一组此类策略，并在需要同样的加密选项和协议时重新将这些策略用于流式处理定位符。 媒体服务帐户具有对应于 StreamingPolicy 条目数的配额。 不应为每个流式处理定位符创建新的 StreamingPolicy。

```c#
private static async Task<StreamingLocator> CreateStreamingLocatorAsync(
    IAzureMediaServicesClient client,
    string resourceGroup,
    string accountName,
    string assetName,
    string locatorName,
    string contentPolicyName)
{
    StreamingLocator locator = await client.StreamingLocators.CreateAsync(
        resourceGroup,
        accountName,
        locatorName,
        new StreamingLocator
        {
            AssetName = assetName,
            StreamingPolicyName = PredefinedStreamingPolicy.ClearKey,
            DefaultContentKeyPolicyName = contentPolicyName
        });

    return locator;
}
```

## <a name="get-a-test-token"></a>获取测试令牌

本教程在内容密钥策略中指定使用令牌限制。 令牌限制策略必须附带由安全令牌服务 (STS) 颁发的令牌。 媒体服务支持采用 [JWT](https://docs.microsoft.com/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_3) 格式的令牌，我们在示例中配置了此格式。

**内容密钥策略**中使用了 ContentKeyIdentifierClaim，这意味着，提供给密钥传送服务的令牌必须包含内容密钥的标识符。 本示例未指定内容密钥，在创建流定位器时，系统创建了一个随机内容密钥。 若要生成测试令牌，必须获取要放入 ContentKeyIdentifierClaim 声明中的 ContentKeyId。

```c#
private static string GetTokenAsync(string issuer, string audience, string keyIdentifier, byte[] tokenVerificationKey)
{
    var tokenSigningKey = new SymmetricSecurityKey(tokenVerificationKey);

    SigningCredentials cred = new SigningCredentials(
        tokenSigningKey,
        // Use the  HmacSha256 and not the HmacSha256Signature option, or the token will not work!
        SecurityAlgorithms.HmacSha256,
        SecurityAlgorithms.Sha256Digest);

    Claim[] claims = new Claim[]
    {
        new Claim(ContentKeyPolicyTokenClaim.ContentKeyIdentifierClaim.ClaimType, keyIdentifier)
    };

    JwtSecurityToken token = new JwtSecurityToken(
        issuer: issuer,
        audience: audience,
        claims: claims,
        notBefore: DateTime.Now.AddMinutes(-5),
        expires: DateTime.Now.AddMinutes(60),
        signingCredentials: cred);

    JwtSecurityTokenHandler handler = new JwtSecurityTokenHandler();

    return handler.WriteToken(token);
}
```

## <a name="build-a-dash-streaming-url"></a>生成 DASH 流 URL

创建[流式处理定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)后，即可获取流式处理 URL。 若要生成 URL，需要连接 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/streamingendpoints) 主机名称和流定位器路径****。 此示例使用默认的**流式处理终结点**。 首次创建媒体服务帐户时，此默认的**流式处理终结点**处于停止状态，因此需要调用 **Start**。

```c#
private static async Task<string> GetDASHStreamingUrlAsync(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string locatorName)
{
    const string DefaultStreamingEndpointName = "default";

    string dashPath = "";

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

        // Look for just the DASH path and generate a URL for the Azure Media Player to playback the content with the AES token to decrypt.
        // Note that the JWT token is set to expire in 1 hour. 
        if (path.StreamingProtocol == StreamingPolicyStreamingProtocol.Dash)
        {
            uriBuilder.Path = path.Paths[0];

            dashPath = uriBuilder.ToString();

        }
    }

    return dashPath;
}
```

## <a name="clean-up-resources-in-your-media-services-account"></a>清理媒体服务帐户中的资源

一般来说，除了打算重用的对象之外，应该清除所有对象（通常，将重用转换、流式处理定位符等）。 如果希望帐户在试验后保持干净状态，则删除不打算重复使用的资源。 例如，以下代码可删除作业、已创建的资产和内容密钥策略：

```c#
private static async Task CleanUpAsync(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string transformName,
    string contentKeyPolicyName)
{

    var jobs = await client.Jobs.ListAsync(resourceGroupName, accountName, transformName);
    foreach (var job in jobs)
    {
        await client.Jobs.DeleteAsync(resourceGroupName, accountName, transformName, job.Name);
    }

    var assets = await client.Assets.ListAsync(resourceGroupName, accountName);
    foreach (var asset in assets)
    {
        await client.Assets.DeleteAsync(resourceGroupName, accountName, asset.Name);
    }

    client.ContentKeyPolicies.Delete(resourceGroupName, accountName, contentKeyPolicyName);

}
```

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组中的任何一个资源（包括为本教程创建的媒体服务和存储帐户），请删除之前创建的资源组。

执行以下 CLI 命令：

```azurecli
az group delete --name amsResourceGroup
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用 DRM 提供保护](protect-with-drm.md)
