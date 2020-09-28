---
title: Azure 媒体服务 DRM 加密和许可证传送服务
titleSuffix: Azure Media Services
description: 了解如何使用 DRM 动态加密和许可传递服务来传送通过 Microsoft PlayReady 或 Apple FairPlay 许可证加密的流。
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
ms.custom: seodec18
ms.openlocfilehash: ba676cbd730ab2719b500efe6f2749ae627d8a60
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91244980"
---
# <a name="tutorial-use-drm-dynamic-encryption-and-license-delivery-service"></a>教程：使用 DRM 动态加密和许可证传送服务

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

> [!NOTE]
> Google Widevine 内容保护服务目前在 Azure 中国区域不可用。

> [!NOTE]
> 尽管本教程使用了 [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.models.liveevent?view=azure-dotnet) 示例，但 [REST API](https://docs.microsoft.com/rest/api/media/liveevents)、[CLI](https://docs.microsoft.com/cli/azure/ams/live-event?view=azure-cli-latest) 或其他受支持的 [SDK](media-services-apis-overview.md#sdks) 的常规步骤是相同的。

可以使用 Azure 媒体服务来传送通过 Microsoft PlayReady 或 Apple FairPlay 许可证加密的流。 如需深入说明，请参阅[使用动态加密保护内容](content-protection-overview.md)。

媒体服务还提供用于传送 PlayReady 和 FairPlay DRM 许可证的服务。 当用户请求受 DRM 保护的内容时，播放器应用会从媒体服务许可证服务请求许可证。 如果播放器应用获得授权，媒体服务许可证服务会向该播放器颁发许可证。 许可证包含客户端播放器用来对内容进行解密和流式传输的解密密钥。

本文基于[使用 DRM 进行加密](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM)示例。

本文中所述的示例生成以下结果：

![Azure Media Player 中的具有 DRM 保护视频的 AMS](./media/protect-with-drm/ams_player.png)

本教程介绍了如何：

> [!div class="checklist"]
> * 创建编码转换。
> * 设置用于验证令牌的签名密钥。
> * 设置对内容密钥策略的要求。
> * 使用指定的流式处理策略创建 StreamingLocator。
> * 创建一个用于播放文件的 URL。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

以下项目是完成本教程所需具备的条件：

* 查看[内容保护概述](content-protection-overview.md)一文。
* 查看[设计带访问控制的多 DRM 内容保护系统](design-multi-drm-system-with-access-control.md)。
* 安装 Visual Studio Code 或 Visual Studio。
* 按照[本快速入门](./create-account-howto.md)所述，创建新的 Azure 媒体服务帐户。
* 根据[访问 API](./access-api-howto.md) 中所述，获取使用媒体服务 API 时所需的凭据
* 在应用配置文件 (appsettings.json) 中设置相应的值。

## <a name="download-code"></a>下载代码

使用以下命令，将包含本文中所述完整 .NET 示例的 GitHub 存储库克隆到计算机：

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```
 
“使用 DRM 进行加密”示例位于 [EncryptWithDRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM) 文件夹中。

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

输出[资产](assets-concept.md)会存储作业编码的结果。  

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

创建新实例时，需要指定希望生成的输出内容[转换](transforms-jobs-concept.md)。 所需参数是 `transformOutput` 对象，如以下代码所示。 每个 TransformOutput 包含一个预设。 预设介绍了视频和/或音频处理操作的分步说明，这些操作将用于生成所需的 TransformOutput。 本文中的示例使用名为 AdaptiveStreaming 的内置预设。 此预设将输入的视频编码为基于输入的分辨率和比特率自动生成的比特率阶梯（比特率 - 分辨率对），并通过与每个比特率 - 分辨率对相对应的 H.264 视频和 AAC 音频生成 ISO MP4 文件。 

在创建新的**转换**之前，应该先检查是否已存在使用 **Get** 方法的转换，如以下代码中所示。  在 Media Services v3**获取**实体上的方法返回**null**如果实体不存在 （不区分大小写的名称检查）。

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

如上所述，**转换**对象为脚本，[作业则是](transforms-jobs-concept.md)对媒体服务的实际请求，请求将转换应用到给定输入视频或音频内容。 Job 指定输入视频位置和输出位置等信息。

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

该作业需要一些时间才能完成操作。 在该过程中，你应能够接收通知。 以下代码示例显示如何轮询服务以获取**作业**状态。

**作业**通常会经历以下状态：**已计划**、**已排队**、**正在处理**、**已完成**（最终状态）。 如果作业出错，则显示“错误”状态  。 如果作业正处于取消过程中，则显示“正在取消”，完成时则显示“已取消”   。

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

内容密钥提供对资产的安全访问。 通过 DRM 加密内容时，需要创建[内容密钥策略](content-key-policy-concept.md)。 此策略配置如何将内容密钥传送到最终的客户端。 内容密钥与流定位器相关联。 媒体服务还提供密钥传送服务，将加密密钥和许可证传送给已授权的用户。

需要在使用指定的配置传送密钥时必须满足的内容密钥策略中设置要求（限制）。 此示例设置了以下配置和要求：

* 配置

    配置了 [PlayReady](playready-license-template-overview.md) 许可证，因此只能由媒体服务许可证传送服务传送这些许可证。 尽管此示例应用未配置 [FairPlay](fairplay-license-overview.md) 许可证，但它包含一个可用来配置 FairPlay 的方法。 可以添加 FairPlay 配置作为另一个选项。

* 限制

    该应用在策略中设置了 JSON Web 令牌 (JWT) 令牌类型限制。

当播放器请求流时，媒体服务将使用指定的密钥动态加密内容。 为解密流，播放器从密钥传送服务请求密钥。 为了确定是否已授权用户获取密钥，服务将评估你为密钥指定的内容密钥策略。

```c#
private static async Task<ContentKeyPolicy> GetOrCreateContentKeyPolicyAsync(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string contentKeyPolicyName,
    byte[] tokenSigningKey)
{
    ContentKeyPolicy policy = await client.ContentKeyPolicies.GetAsync(resourceGroupName, accountName, contentKeyPolicyName);

    if (policy == null)
    {
        ContentKeyPolicySymmetricTokenKey primaryKey = new ContentKeyPolicySymmetricTokenKey(tokenSigningKey);
        List<ContentKeyPolicyTokenClaim> requiredClaims = new List<ContentKeyPolicyTokenClaim>()
        {
            ContentKeyPolicyTokenClaim.ContentKeyIdentifierClaim
        };
        List<ContentKeyPolicyRestrictionTokenKey> alternateKeys = null;
        ContentKeyPolicyTokenRestriction restriction 
            = new ContentKeyPolicyTokenRestriction(Issuer, Audience, primaryKey, ContentKeyPolicyRestrictionTokenType.Jwt, alternateKeys, requiredClaims);

        ContentKeyPolicyPlayReadyConfiguration playReadyConfig = ConfigurePlayReadyLicenseTemplate();
        // ContentKeyPolicyFairPlayConfiguration fairplayConfig = ConfigureFairPlayPolicyOptions();

        List<ContentKeyPolicyOption> options = new List<ContentKeyPolicyOption>();

        options.Add(
            new ContentKeyPolicyOption()
            {
                Configuration = playReadyConfig,
                // If you want to set an open restriction, use
                // Restriction = new ContentKeyPolicyOpenRestriction()
                Restriction = restriction
            });

     // add CBCS ContentKeyPolicyOption into the list
     //   options.Add(
     //       new ContentKeyPolicyOption()
     //       {
     //           Configuration = fairplayConfig,
     //           Restriction = restriction,
     //           Name = "ContentKeyPolicyOption_CBCS"
     //       });

        policy = await client.ContentKeyPolicies.CreateOrUpdateAsync(resourceGroupName, accountName, contentKeyPolicyName, options);
    }
    else
    {
        // Get the signing key from the existing policy.
        var policyProperties = await client.ContentKeyPolicies.GetPolicyPropertiesWithSecretsAsync(resourceGroupName, accountName, contentKeyPolicyName);
        var restriction = policyProperties.Options[0].Restriction as ContentKeyPolicyTokenRestriction;
        if (restriction != null)
        {
            var signingKey = restriction.PrimaryVerificationKey as ContentKeyPolicySymmetricTokenKey;
            if (signingKey != null)
            {
                TokenSigningKey = signingKey.KeyValue;
            }
        }
    }
    return policy;
}
```

## <a name="create-a-streaming-locator"></a>创建流定位符

完成编码并设置内容密钥策略后，下一步是使输出资产中的视频可供客户端播放。 可以通过两个步骤来提供视频：

1. 创建[流式处理定位符](streaming-locators-concept.md)。
2. 生成客户端可以使用的流式处理 URL。

创建**流定位器**的过程称为发布。 默认情况下，除非配置可选的开始和结束时间，否则调用 API 后，流式处理定位符**** 立即生效， 并持续到被删除为止。

创建**流定位器**时，需要指定所需的 `StreamingPolicyName`。 本教程使用某个预定义的流式处理策略来告知 Azure 媒体服务如何发布流式处理的内容。 在此示例中，请将 StreamingLocator.StreamingPolicyName 设置为“Predefined_MultiDrmCencStreaming”策略。 将应用 PlayReady 加密，并根据配置的 DRM 许可证将密钥传送到播放客户端。 如果还要使用 CBCS (FairPlay) 加密流，请使用“Predefined_MultiDrmStreaming”。

> [!IMPORTANT]
> 使用自定义的[流策略](streaming-policy-concept.md)时，应为媒体服务帐户设计有限的一组此类策略，并在需要同样的加密选项和协议时重新将这些策略用于 StreamingLocators。 媒体服务帐户具有对应于 StreamingPolicy 条目数的配额。 不应为每个 StreamingLocator 创建新的 StreamingPolicy。

```c#
private static async Task<StreamingLocator> CreateStreamingLocatorAsync(
    IAzureMediaServicesClient client,
    string resourceGroup,
    string accountName,
    string assetName,
    string locatorName,
    string contentPolicyName)
{
    // If you also added FairPlay, use "Predefined_MultiDrmStreaming
    StreamingLocator locator = await client.StreamingLocators.CreateAsync(
        resourceGroup,
        accountName,
        locatorName,
        new StreamingLocator
        {
            AssetName = assetName,
            // "Predefined_MultiDrmCencStreaming" policy supports envelope and cenc encryption
            // And sets two content keys on the StreamingLocator
            StreamingPolicyName = "Predefined_MultiDrmCencStreaming",
            DefaultContentKeyPolicyName = contentPolicyName
        });

    return locator;
}
```

## <a name="get-a-test-token"></a>获取测试令牌

本教程在内容密钥策略中指定使用令牌限制。 令牌限制策略必须附带由安全令牌服务 (STS) 颁发的令牌。 媒体服务支持采用 [JWT](https://docs.microsoft.com/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_3) 格式的令牌，我们在示例中配置了此格式。

ContentKeyPolicy 中使用了 ContentKeyIdentifierClaim，这意味着，提供给密钥传送服务的令牌必须包含 ContentKey 的标识符。 本示例未指定内容密钥，在创建流定位器时，系统会创建一个随机内容密钥。 若要生成测试令牌，必须获取要放入 ContentKeyIdentifierClaim 声明中的 ContentKeyId。

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
    
    // To set a limit on how many times the same token can be used to request a key or a license.
    // add  the "urn:microsoft:azure:mediaservices:maxuses" claim.
    // For example, claims.Add(new Claim("urn:microsoft:azure:mediaservices:maxuses", 4));
    
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

## <a name="build-a-streaming-url"></a>生成流 URL

创建 [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators) 后，可以获取流 URL。 若要生成 URL，需要连接 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/streamingendpoints) 主机名称和流定位器路径****。 此示例使用默认的**流式处理终结点**。 首次创建媒体服务帐户时，此默认的**流式处理终结点**处于停止状态，因此需要调用 **Start**。

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

        // Look for just the DASH path and generate a URL for the Azure Media Player to playback the encrypted DASH content. 
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

运行应用时看到以下屏幕：

![使用 DRM 提供保护](./media/protect-with-drm/playready_encrypted_url.png)

可以打开浏览器并粘贴生成的 URL 来启动 Azure Media Player 演示页，其中已经填充了该 URL 和令牌。

## <a name="clean-up-resources-in-your-media-services-account"></a>清理媒体服务帐户中的资源

一般来说，除了打算重用的对象之外，应该清除所有对象（通常，将重用转换、StreamingLocators 等）。 如果希望帐户在试验后保持干净状态，则删除不打算重复使用的资源。 例如，以下代码可删除作业、已创建的资产和内容密钥策略：

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

    var streamingLocators = await client.StreamingLocators.ListAsync(resourceGroupName, accountName);
    foreach (var locator in streamingLocators)
    {
        await client.StreamingLocators.DeleteAsync(resourceGroupName, accountName, locator.Name);
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

签出

> [!div class="nextstepaction"]
> [使用 AES-128 进行保护](protect-with-aes128.md)
