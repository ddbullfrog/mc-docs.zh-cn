---
title: 从 HTTPS URL 创建 Azure 媒体服务作业输入 | Microsoft Docs
description: 本主题演示如何从 HTTPS URL 创建 Azure 媒体服务作业输入。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
origin.date: 08/31/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.openlocfilehash: 1761385125a2d97a3f0d8d6de236607b1f3911f3
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245607"
---
# <a name="create-a-job-input-from-an-https-url"></a>从 HTTPS URL 创建作业输入

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

在媒体服务 v3 中提交作业来处理视频时，必须告知媒体服务查找输入视频的位置。 其中一个选项是指定 HTTPS URL 作为作业输入（如本示例所示）。 请注意，目前，AMS v3 不支持基于 HTTPS URL 的块传输编码。 有关完整示例，请参阅此 [GitHub 示例](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/blob/master/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs)。

> [!TIP]
> 在开始开发之前，请查看[使用媒体服务 v3 API 进行开发](media-services-apis-overview.md)（包括有关访问 API、命名约定等的信息）

## <a name="net-sample"></a>.NET 示例

以下代码说明了如何使用 HTTPS URL 输入创建作业。

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
    // to get the existing job. In Media Services v3, Get methods on entities returns null 
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

## <a name="job-error-codes"></a>作业错误代码

请参阅[错误代码](https://docs.microsoft.com/rest/api/media/jobs/get#joberrorcode)。

## <a name="next-steps"></a>后续步骤

[从本地文件创建作业输入](job-input-from-local-file-how-to.md)。
