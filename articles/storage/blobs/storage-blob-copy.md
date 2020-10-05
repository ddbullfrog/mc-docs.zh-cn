---
title: 使用 Azure 存储 API 复制 blob
description: 了解如何使用 Azure 存储客户端库复制 blob。
services: storage
author: WenJason
ms.author: v-jay
origin.date: 09/10/2020
ms.date: 09/28/2020
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 7b52ea56c7548f6fd01ffd1829666dbdcd3c9749
ms.sourcegitcommit: 119a3fc5ffa4768b1bd8202191091bd4d873efb4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2020
ms.locfileid: "91026544"
---
# <a name="copy-a-blob-with-azure-storage-client-libraries"></a>使用 Azure 存储客户端库复制 blob

本文演示如何在 Azure 存储帐户中复制 blob。 此外介绍如何中止异步复制操作。 示例代码使用 Azure 存储客户端库。

## <a name="about-copying-blobs"></a>关于复制 Blob

复制同一存储帐户中的 Blob 属于同步操作。 跨帐户复制属于异步操作。

复制操作的源 Blob 可以是块 Blob、追加 Blob、页 Blob 或快照。 如果目标 Blob 已存在，该 Blob 的类型必须与源 Blob 的类型相同。 将覆盖现有的目标 Blob。

当复制操作正在进行时，无法修改目标 Blob。 目标 Blob 只能有一个未完成的复制操作。 换言之，一个 Blob 不能是多个挂起的复制操作的目标。

始终复制整个源 Blob 或文件。 不支持复制字节范围或块集。

复制 Blob 时，会将系统属性复制到具有相同值的目标 Blob。

复制操作可以采用以下任一形式：

- 将源 Blob 复制到具有不同名称的目标 Blob。 目标 Blob 可以是相同类型（块、追加或页 Blob）的现有 Blob，也可以是复制操作创建的新 Blob。
- 将源 Blob 复制到具有相同名称的目标 Blob，从而有效替换目标 Blob。 此类复制操作会删除所有未提交的块，并覆盖目标 Blob 的元数据。
- 将 Azure 文件服务中的源文件复制到目标 Blob。 目标 Blob 可以是现有的块 Blob，也可以是复制操作创建的新块 Blob。 不支持从文件复制到页 Blob 或追加 Blob。
- 将快照复制到其基本 Blob 上。 通过将快照提升到基本 Blob 的位置，可还原早期版本的 Blob。
- 将快照复制到具有不同名称的目标 Blob。 生成的目标 Blob 是可写的 Blob，而不是快照。

## <a name="copy-a-blob"></a>复制 Blob

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

若要复制 Blob，请调用以下方法之一：

- [StartCopyFromUri](https://docs.microsoft.com/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.startcopyfromuri)
- [StartCopyFromUriAsync](https://docs.microsoft.com/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.startcopyfromuriasync)

StartCopyFromUri 和 StartCopyFromUriAsync 方法返回包含有关复制操作的信息的 [CopyFromUriOperation](https://docs.microsoft.com/dotnet/api/azure.storage.blobs.models.copyfromurioperation) 对象。

以下代码示例获取表示以前创建的 Blob 的 [BlobClient](https://docs.microsoft.com/dotnet/api/azure.storage.blobs.blobclient)，并将其复制到同一容器中的新 Blob：

```csharp
private static async Task CopyBlobAsync(BlobContainerClient container)
{
    try
    {
        // Get the name of the first blob in the container to use as the source.
        string blobName = container.GetBlobs().FirstOrDefault().Name;

        // Create a BlobClient representing the source blob to copy.
        BlobClient sourceBlob = container.GetBlobClient(blobName);

        // Ensure that the source blob exists.
        if (await sourceBlob.ExistsAsync())
        {
            // Lease the source blob for the copy operation 
            // to prevent another client from modifying it.
            BlobLeaseClient lease = sourceBlob.GetBlobLeaseClient();

            // Specifying -1 for the lease interval creates an infinite lease.
            await lease.AcquireAsync(TimeSpan.FromSeconds(-1));

            // Get the source blob's properties and display the lease state.
            BlobProperties sourceProperties = await sourceBlob.GetPropertiesAsync();
            Console.WriteLine($"Lease state: {sourceProperties.LeaseState}");

            // Get a BlobClient representing the destination blob with a unique name.
            BlobClient destBlob = 
                container.GetBlobClient(Guid.NewGuid() + "-" + sourceBlob.Name);

            // Start the copy operation.
            await destBlob.StartCopyFromUriAsync(sourceBlob.Uri);

            // Get the destination blob's properties and display the copy status.
            BlobProperties destProperties = await destBlob.GetPropertiesAsync();

            Console.WriteLine($"Copy status: {destProperties.CopyStatus}");
            Console.WriteLine($"Copy progress: {destProperties.CopyProgress}");
            Console.WriteLine($"Completion time: {destProperties.CopyCompletedOn}");
            Console.WriteLine($"Total bytes: {destProperties.ContentLength}");

            // Update the source blob's properties.
            sourceProperties = await sourceBlob.GetPropertiesAsync();

            if (sourceProperties.LeaseState == LeaseState.Leased)
            {
                // Break the lease on the source blob.
                await lease.BreakAsync();

                // Update the source blob's properties to check the lease state.
                sourceProperties = await sourceBlob.GetPropertiesAsync();
                Console.WriteLine($"Lease state: {sourceProperties.LeaseState}");
            }
        }
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine(ex.Message);
        Console.ReadLine();
        throw;
    }
}
```

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

若要复制 Blob，请调用以下方法之一：

- [StartCopy](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblob.startcopy?view=azure-dotnet)
- [StartCopyAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblob.startcopyasync?view=azure-dotnet)

**StartCopy** 和 **StartCopyAsync** 方法返回用于检查状态或中止复制操作的复制 ID 值。

以下代码示例获取对以前创建的 Blob 的引用，并将其复制到同一容器中的新 Blob：

```csharp
private static async Task CopyBlockBlobAsync(CloudBlobContainer container)
{
    CloudBlockBlob sourceBlob = null;
    CloudBlockBlob destBlob = null;
    string leaseId = null;

    try
    {
        // Get a block blob from the container to use as the source.
        sourceBlob = container.ListBlobs().OfType<CloudBlockBlob>().FirstOrDefault();

        // Lease the source blob for the copy operation 
        // to prevent another client from modifying it.
        // Specifying null for the lease interval creates an infinite lease.
        leaseId = await sourceBlob.AcquireLeaseAsync(null);

        // Get a reference to a destination blob (in this case, a new blob).
        destBlob = container.GetBlockBlobReference("copy of " + sourceBlob.Name);

        // Ensure that the source blob exists.
        if (await sourceBlob.ExistsAsync())
        {
            // Get the ID of the copy operation.
            string copyId = await destBlob.StartCopyAsync(sourceBlob);

            // Fetch the destination blob's properties before checking the copy state.
            await destBlob.FetchAttributesAsync();

            Console.WriteLine("Status of copy operation: {0}", destBlob.CopyState.Status);
            Console.WriteLine("Completion time: {0}", destBlob.CopyState.CompletionTime);
            Console.WriteLine("Bytes copied: {0}", destBlob.CopyState.BytesCopied.ToString());
            Console.WriteLine("Total bytes: {0}", destBlob.CopyState.TotalBytes.ToString());
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
    finally
    {
        // Break the lease on the source blob.
        if (sourceBlob != null)
        {
            await sourceBlob.FetchAttributesAsync();

            if (sourceBlob.Properties.LeaseState != LeaseState.Available)
            {
                await sourceBlob.BreakLeaseAsync(new TimeSpan(0));
            }
        }
    }
}
```

# <a name="python-v12"></a>[Python v12](#tab/python)

若要复制 Blob，请调用 [start_copy_from_url](https://docs.microsoft.com/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient#start-copy-from-url-source-url--metadata-none--incremental-copy-false----kwargs-) 方法。 start_copy_from_url 方法返回包含有关复制操作的信息的字典。

以下代码示例获取表示以前创建的 Blob 的 [BlobClient](https://docs.microsoft.com/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient)，并将其复制到同一容器中的新 Blob：

```Python
def blob_copy(self, container_name, blob_name):

    # Create a BlobClient from a connection string
    # retrieved from an environment variable named
    # AZURE_STORAGE_CONNECTION_STRING
    source_blob = BlobClient.from_connection_string(
        os.getenv("AZURE_STORAGE_CONNECTION_STRING"), 
        container_name, blob_name
        )

    try:
        # Lease the source blob for the copy operation
        # to prevent another client from modifying it.
        lease = BlobLeaseClient(source_blob)
        lease.acquire()

        # Get the source blob's properties and display the lease state.
        source_props = source_blob.get_blob_properties()
        print("Lease state: " + source_props.lease.state)

        # Create a BlobClient representing the
        # destination blob with a unique name.
        dest_blob = BlobClient.from_connection_string(
            os.getenv("AZURE_STORAGE_CONNECTION_STRING"),
            container_name, str(uuid.uuid4()) + "-" + blob_name
            )

        # Start the copy operation.
        dest_blob.start_copy_from_url(source_blob.url)

        # Get the destination blob's properties to check the copy status.
        properties = dest_blob.get_blob_properties()
        copy_props = properties.copy

        # Display the copy status
        print("Copy status: " + copy_props["status"])
        print("Copy progress: " + copy_props["progress"]);
        print("Completion time: " + str(copy_props["completion_time"]));
        print("Total bytes: " + str(properties.size));

        if (source_props.lease.state == "leased"):
            # Break the lease on the source blob.
            lease.break_lease()

            # Update the destination blob's properties to check the lease state.
            source_props = source_blob.get_blob_properties()
            print("Lease state: " + source_props.lease.state)

    except ResourceNotFoundError as ex:
        print("ResourceNotFoundError: ", ex.message)

    except ServiceRequestError as ex:
        print("ServiceRequestError: ", ex.message)
```

---

## <a name="abort-a-copy-operation"></a>中止复制操作

中止复制操作会导致目标 Blob 长度为零。 但是，目标 Blob 的元数据将包含从源 Blob 复制的新值，或者在复制操作过程中显式设置的新值。 若要在复制之前保留原始元数据，请在调用一个复制方法之前创建目标 Blob 的快照。

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

检查目标 Blob 中的 [BlobProperties.CopyStatus](https://docs.microsoft.com/dotnet/api/azure.storage.blobs.models.blobproperties.copystatus) 属性，以获取复制操作的状态。 完成复制时，将提交最终的 Blob。

如果中止复制操作，目标 Blob 的复制状态将设置为 [CopyStatus.Aborted](/dotnet/api/microsoft.windowsazure.storage.blob.copystatus)。

[AbortCopyFromUri](https://docs.microsoft.com/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.abortcopyfromuri) 和 [AbortCopyFromUriAsync](https://docs.microsoft.com/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.abortcopyfromuriasync) 方法会取消正在进行的复制操作。

```csharp
// Get the destination blob's properties to check the copy status.
BlobProperties destProperties = destBlob.GetProperties();

// Check the copy status. If the status is pending, abort the copy operation.
if (destProperties.CopyStatus == CopyStatus.Pending)
{
    await destBlob.AbortCopyFromUriAsync(destProperties.CopyId);
    Console.WriteLine($"Copy operation {destProperties.CopyId} has been aborted.");
}
```

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

检查目标 Blob 中的 [CopyState.Status](/dotnet/api/microsoft.windowsazure.storage.blob.copystate.status) 属性，以获取复制操作的状态。 完成复制时，将提交最终的 Blob。

如果中止复制操作，目标 Blob 的复制状态将设置为 [CopyStatus.Aborted](/dotnet/api/microsoft.windowsazure.storage.blob.copystatus)。

[AbortCopy](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblob.abortcopy) 和 [AbortCopyAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblob.abortcopyasync) 方法会取消正在进行的复制操作。

```csharp
// Fetch the destination blob's properties before checking the copy state.
await destBlob.FetchAttributesAsync();

// Check the copy status. If it is still pending, abort the copy operation.
if (destBlob.CopyState.Status == CopyStatus.Pending)
{
    await destBlob.AbortCopyAsync(copyId);
    Console.WriteLine("Copy operation {0} has been aborted.", copyId);
}
```

# <a name="python-v12"></a>[Python v12](#tab/python)

检查 [get_blob_properties](https://docs.microsoft.com/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient#get-blob-properties---kwargs-) 方法返回的 [CopyProperties](https://docs.microsoft.com/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.copyproperties) 字典中的“status”条目，以获取复制操作的状态。 完成复制时，将提交最终的 Blob。

如果中止复制操作，[status](https://docs.microsoft.com/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.copyproperties) 将设置为“aborted”。

[abort_copy](https://docs.microsoft.com/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient#abort-copy-copy-id----kwargs-) 方法会取消正在进行的复制操作。

```Python
# Get the destination blob's properties to check the copy status.
properties = dest_blob.get_blob_properties()
copy_props = properties.copy

# Check the copy status. If the status is pending, abort the copy operation.
if (copy_props["status"] == "pending"):
    dest_blob.abort_copy(copy_props["id"])
    print("Copy operation " + copy_props["id"] + " has been aborted.")
```

---

## <a name="azure-sdks"></a>Azure SDK

获取有关 Azure SDK 的详细信息：

 - [用于 .NET 的 Azure SDK](https://github.com/azure/azure-sdk-for-net)
 - [Azure SDK for Java](https://github.com/azure/azure-sdk-for-java)
 - [Azure SDK for Python](https://github.com/azure/azure-sdk-for-python)
 - [Azure SDK for JavaScript](https://github.com/azure/azure-sdk-for-js)

## <a name="next-steps"></a>后续步骤

以下主题包含有关使用 Azure REST API 复制 Blob 和中止正在进行的复制操作的信息。

- [复制 Blob](https://docs.microsoft.com/rest/api/storageservices/copy-blob)
- [中止复制 Blob](https://docs.microsoft.com/rest/api/storageservices/abort-copy-blob)
