---
title: 从本地文件创建 Azure 媒体服务作业输入 | Microsoft Docs
description: 本文演示如何从本地文件创建 Azure 媒体服务作业输入。
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
ms.openlocfilehash: 249dd54799ad405b4fb26469168e77b8e408f447
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245606"
---
# <a name="create-a-job-input-from-a-local-file"></a>从本地文件创建作业输入

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

在媒体服务 v3 中提交作业来处理视频时，必须告知媒体服务查找输入视频的位置。 可将输入视频存储为媒体服务资产，这种情况下会基于文件（存储在本地或 Azure Blob 存储中）创建输入资产。 本主题介绍如何从本地文件创建作业输入。 有关完整示例，请参阅此 [GitHub 示例](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs)。

## <a name="prerequisites"></a>必备条件 

* [创建媒体服务帐户](./create-account-howto.md)。

## <a name="net-sample"></a>.NET 示例

以下代码演示如何创建输入资产并将其用作作业的输入。 此 CreateInputAsset 函数执行以下操作：

* 创建资产
* 获取资产的[存储中容器](../../storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-a-container)的可写 [SAS URL](../../storage/common/storage-sas-overview.md)
* 使用 SAS URL 将文件上传到存储中的容器中

```c#
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

以下代码片段将创建一个输出资产（如果它尚不存在）：

```c#
private static async Task<Asset> CreateOutputAssetAsync(
    IAzureMediaServicesClient client, 
    string resourceGroupName, 
    string accountName, 
    string assetName)
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

以下代码片段将提交编码作业：

```c#
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

## <a name="job-error-codes"></a>作业错误代码

请参阅[错误代码](https://docs.microsoft.com/rest/api/media/jobs/get#joberrorcode)。

## <a name="next-steps"></a>后续步骤

[从 HTTPS URL 创建作业输入](job-input-from-http-how-to.md)。
