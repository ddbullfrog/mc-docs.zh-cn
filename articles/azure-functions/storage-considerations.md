---
title: Azure Functions 的存储注意事项
description: 了解 Azure Functions 的要求和存储数据加密。
ms.topic: conceptual
ms.date: 08/12/2020
ms.openlocfilehash: e45fee8cf1946dc1f2a3eecc42f4e6f312de4862
ms.sourcegitcommit: 84606cd16dd026fd66c1ac4afbc89906de0709ad
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/14/2020
ms.locfileid: "88222582"
---
# <a name="storage-considerations-for-azure-functions"></a>Azure Functions 的存储注意事项

创建函数应用实例时，Azure Functions 需要 Azure 存储帐户。 函数应用可以使用以下存储服务：


|存储服务  | 函数用法  |
|---------|---------|
| [Azure Blob 存储](../storage/blobs/storage-blobs-introduction.md)     | 维护绑定状态和函数密钥。  <br/>还由 [Durable Functions 中的任务中心](durable/durable-functions-task-hubs.md)使用。 |
| [Azure 文件](../storage/files/storage-files-introduction.md)  | 用于在[消耗计划](functions-scale.md#consumption-plan)中存储和运行函数应用代码的文件共享。 |
| [Azure 队列存储](../storage/queues/storage-queues-introduction.md)     | 由 [Durable Functions 中的任务中心](durable/durable-functions-task-hubs.md)使用。   |
| [Azure 表存储](../storage/tables/table-storage-overview.md)  |  由 [Durable Functions 中的任务中心](durable/durable-functions-task-hubs.md)使用。       |

> [!IMPORTANT]
> 使用消耗托管计划时，函数代码和绑定配置文件存储在主存储帐户的 Azure 文件存储中。 删除主存储帐户时，此内容将随之删除且无法恢复。

## <a name="storage-account-requirements"></a>存储帐户要求

创建函数应用时，必须创建或链接到支持 Blob、队列和表存储的常规用途的 Azure 存储帐户。 这是因为 Functions 依赖于 Azure 存储，执行管理触发器和记录函数执行等操作。 某些存储帐户不支持队列和表。 这些帐户包括仅限 blob 的存储帐户、Azure 高级存储和使用 ZRS 复制的常规用途存储帐户。 创建函数应用时，将从“存储帐户”边栏选项卡中筛选出这些不支持的帐户。

若要了解有关存储帐户类型的详细信息，请参阅 [Azure 存储服务简介](../storage/common/storage-introduction.md#core-storage-services)。 

虽然可以将现有存储帐户用于函数应用，不过必须确保它满足这些要求。 作为函数应用创建流的一部分创建的存储帐户可保证满足这些存储帐户要求。  

## <a name="storage-account-guidance"></a>存储帐户指导

每个 Function App 都需要存储帐户才能运行。 如果该帐户已删除，则你的函数应用将不会运行。 若要对存储相关问题进行故障排除，请参阅[如何对存储相关问题进行故障排除](functions-recover-storage-account.md)。 以下附加注意事项适用于函数应用使用的存储帐户。

### <a name="storage-account-connection-setting"></a>存储帐户连接设置

存储帐户连接在 [AzureWebJobsStorage 应用程序设置](./functions-app-settings.md#azurewebjobsstorage)中进行维护。 

重新生成存储密钥时，必须更新存储帐户连接字符串。 [在此处阅读有关存储密钥管理的详细信息](../storage/common/storage-account-create.md)。

### <a name="shared-storage-accounts"></a>共享存储帐户

多个函数应用可以共享同一个存储帐户，而不会出现任何问题。 例如，在 Visual Studio 中，可以使用 Azure 存储仿真器开发多个应用。 在这种情况下，仿真器的作用类似于单个存储帐户。 函数应用使用的同一个存储帐户也可用于存储应用程序数据。 但是在生产环境中，这种方法并不总是个好主意。

### <a name="optimize-storage-performance"></a>优化存储性能

[!INCLUDE [functions-shared-storage](../../includes/functions-shared-storage.md)]

## <a name="storage-data-encryption"></a>存储数据加密

[!INCLUDE [functions-storage-encryption](../../includes/functions-storage-encryption.md)]

## <a name="next-steps"></a>后续步骤

详细了解 Azure Functions 托管选项。

> [!div class="nextstepaction"]
> [Azure Functions 的缩放和托管](functions-scale.md)

