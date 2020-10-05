---
title: 如何通过 Node.js 使用 Azure 队列存储 - Azure 存储
description: 学习使用 Azure 队列服务创建和删除队列。 学习使用 Node.js 插入、获取和删除消息。
author: WenJason
ms.author: v-jay
origin.date: 08/31/2020
ms.date: 09/28/2020
ms.service: storage
ms.subservice: queues
ms.topic: how-to
ms.reviewer: dineshm
ms.custom: seo-javascript-september2019, devx-track-javascript
ms.openlocfilehash: 4c102dd9099e740d11f07f27e12f5467349716be
ms.sourcegitcommit: 119a3fc5ffa4768b1bd8202191091bd4d873efb4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2020
ms.locfileid: "91026638"
---
# <a name="how-to-use-azure-queue-storage-from-nodejs"></a>如何通过 Node.js 使用 Azure 队列存储

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

## <a name="overview"></a>概述

本指南演示了如何使用 Azure 队列服务完成常见方案。 相关示例是使用 Node.js API 编写的。 涵盖的方案包括插入、速览、获取和删除队列消息。 还将学习创建和删除队列。

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-nodejs-application"></a>创建 Node.js 应用程序

若要创建空白的 Node.js 应用程序，请参阅[在 Azure 应用服务中创建 Node.js Web 应用][Create a Node.js web app in Azure App Service]、[生成 Node.js 应用程序并将其部署到 Azure 云服务][Build and deploy a Node.js application to an Azure Cloud Service]（使用 Windows PowerShell）或 [Visual Studio Code][Visual Studio Code]。

## <a name="configure-your-application-to-access-storage"></a>配置应用程序以访问存储

[用于 JavaScript 的 Azure 存储客户端库][Azure Storage client library for JavaScript]包括一组便于与存储 REST 服务通信的库。

### <a name="use-node-package-manager-npm-to-obtain-the-package"></a>使用 Node 包管理器 (NPM) 可获取该程序包

1. 使用 PowerShell (Windows)、Terminal (Mac) 或 Bash (Unix) 等命令行界面，导航到在其中创建了示例应用程序的文件夹。

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

1. 在命令窗口中键入 **npm install \@azure/storage-queue**。

1. 验证是否已创建 **node\_modules** 文件夹。 在该文件夹中，你会发现 **\@azure/storage-queue** 包，其中包含访问存储所需的客户端库。

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

1. 在命令窗口中键入 **npm install azure-storage** 。

1. 验证是否已创建 **node\_modules** 文件夹。 在该文件夹中，你会发现 **azure-storage** 包，其中包含访问存储所需的库。

---

### <a name="import-the-package"></a>导入包

使用你的代码编辑器，将以下内容添加到要在其中使用队列的 JavaScript 文件的顶部。

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

```javascript
const { QueueClient, QueueServiceClient } = require("@azure/storage-queue");
```

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

```javascript
var azure = require('azure-storage');
```

---

## <a name="how-to-create-a-queue"></a>如何创建队列

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

以下代码获取名为 `AZURE_STORAGE_CONNECTION_STRING` 的环境变量的值并使用它创建 [QueueServiceClient](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queueserviceclient) 对象。 然后将使用 **QueueServiceClient** 对象创建 [QueueClient](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queueclient) 对象。 可以通过 **QueueClient** 对象处理特定队列。

```javascript
// Retrieve the connection from an environment
// variable called AZURE_STORAGE_CONNECTION_STRING
const connectionString = process.env.AZURE_STORAGE_CONNECTION_STRING;

// Create a unique name for the queue
const queueName = "myqueue-" + Date.now().toString();

console.log("Creating queue: ", queueName);

// Instantiate a QueueServiceClient which will be used
// to create a QueueClient and to list all the queues
const queueServiceClient = new QueueServiceClient.fromConnectionString(connectionString);

// Get a QueueClient which will be used
// to create and manipulate a queue
const queueClient = queueServiceClient.getQueueClient(queueName);

// Create the queue
await queueClient.create();
```

如果队列已存在，则会引发异常。

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Azure 模块将读取环境变量 `AZURE_STORAGE_ACCOUNT`、`AZURE_STORAGE_ACCESS_KEY` 或 `AZURE_STORAGE_CONNECTION_STRING`，以获取连接到 Azure 存储帐户所需的信息。 如果未设置这些环境变量，则在调用 **createQueueService** 时必须指定帐户信息。

以下代码创建一个 **QueueService** 对象，可通过该对象来操作队列。

```javascript
var queueSvc = azure.createQueueService();
```

调用 **createQueueIfNotExists** 方法以创建具有指定名称的新队列；如果该队列已存在，则返回该队列。

```javascript
queueSvc.createQueueIfNotExists('myqueue', function(error, results, response){
  if(!error){
    // Queue created or exists
  }
});
```

如果创建了队列，则 `result.created` 为 true。 如果队列已存在，则 `result.created` 为 false。

---

## <a name="how-to-insert-a-message-into-a-queue"></a>如何在队列中插入消息

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

若要向队列添加消息，请调用 [sendMessage](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queueclient#sendmessage-string--queuesendmessageoptions-) 方法。

```javascript
messageText = "Hello, World";
console.log("Adding message to the queue: ", messageText);

// Add a message to the queue
await queueClient.sendMessage(messageText);
```

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

若要在队列中插入消息，请调用 **createMessage** 方法，以便创建一条新消息并将其添加到队列中。

```javascript
queueSvc.createMessage('myqueue', "Hello world!", function(error, results, response){
  if(!error){
    // Message inserted
  }
});
```

---

## <a name="how-to-peek-at-the-next-message"></a>如何速览下一条消息

可以通过调用 **peekMessages** 方法来速览队列中的消息，而不必将其从队列中删除。

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

默认情况下， [peekMessages](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queueclient#peekmessages-queuepeekmessagesoptions-) 扫视单条消息。 以下示例速览队列中的前五条消息。 如果可见消息少于五条，则只返回这些可见消息。

```javascript
// Peek at messages in the queue
const peekedMessages = await queueClient.peekMessages({ numberOfMessages: 5 });

for (i = 0; i < peekedMessages.peekedMessageItems.length; i++) {
    // Display the peeked message
    console.log("Peeked message: ", peekedMessages.peekedMessageItems[i].messageText);
}
```

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

默认情况下， **peekMessages** 扫视单条消息。

```javascript
queueSvc.peekMessages('myqueue', function(error, results, response){
  if(!error){
    // Message text is in results[0].messageText
  }
});
```

`result` 包含该消息。

---

当队列中没有消息时，调用 **peekMessages** 不会返回错误。 但是，不会返回消息。

## <a name="how-to-change-the-contents-of-a-queued-message"></a>如何更改已排队消息的内容

以下示例将更新消息的文本。

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

通过调用 [updateMessage](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queueclient#updatemessage-string--string--string--number--queueupdatemessageoptions-) 更改已在队列中的消息的内容。 

```javascript
// Get the first message in the queue
var receivedMessages = await queueClient.receiveMessages();
const firstMessage = receivedMessages.receivedMessageItems[0];

// Update the received message
await queueClient.updateMessage(
    firstMessage.messageId,
    firstMessage.popReceipt,
    "This message has been updated"
);
```

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

通过调用 **updateMessage** 在队列中就地更改消息的内容。 

```javascript
queueSvc.getMessages('myqueue', function(error, getResults, getResponse){
  if(!error){
    // Got the message
    var message = getResults[0];
    queueSvc.updateMessage('myqueue', message.messageId, message.popReceipt, 10, {messageText: 'new text'}, function(error, updateResults, updateResponse){
      if(!error){
        // Message updated successfully
      }
    });
  }
});
```

---

## <a name="how-to-dequeue-a-message"></a>如何将消息取消排队

将消息取消排队是一个两阶段过程：

1. 获取消息。

1. 删除消息。

以下示例获取一条消息，然后将其删除。

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

若要获取消息，请调用 [receiveMessages](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queueclient#receivemessages-queuereceivemessageoptions-) 方法。 此调用会使消息在队列中不可见，使其他客户端无法处理它们。 当应用程序处理完某条消息后，即可调用 [deleteMessage](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queueclient#deletemessage-string--string--queuedeletemessageoptions-) 将其从队列中删除。

```javascript
// Get next message from the queue
receivedMessages = await queueClient.receiveMessages();
var message = receivedMessages.receivedMessageItems[0];

console.log("Dequeuing message: ", message.messageText);

await queueClient.deleteMessage(message.messageId, message.popReceipt);
```

默认情况下，消息只会隐藏 30 秒。 30 秒后，其他客户端就会看到它。 你可以通过在调用 **receiveMessages** 时设置 [options.visibilityTimeout](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queuereceivemessageoptions#visibilitytimeout) 来指定其他值。

当队列中没有消息时，调用 **receiveMessages** 不会返回错误。 但是，不会返回消息。

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

若要获取消息，请调用 **getMessages** 方法。 此调用会使消息在队列中不可见，使其他客户端无法处理它们。 当应用程序处理完某条消息后，即可调用 **deleteMessage** 将其从队列中删除。

```javascript
queueSvc.getMessages('myqueue', function(error, results, response){
  if(!error){
    // Message text is in results[0].messageText
    var message = results[0];
    queueSvc.deleteMessage('myqueue', message.messageId, message.popReceipt, function(error, response){
      if(!error){
        //message deleted
      }
    });
  }
});
```

默认情况下，消息只会隐藏 30 秒。 30 秒后，其他客户端就会看到它。 可以将 `options.visibilityTimeout` 与 **getMessages** 配合使用，以指定其他值。

当队列中没有消息时，使用 **getMessages** 不会返回错误。 但是，不会返回消息。

---

## <a name="additional-options-for-dequeuing-messages"></a>用于取消对消息进行排队的其他选项

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

可以通过两种方式自定义队列中的消息检索：

* [options.numberOfMessages](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queuereceivemessageoptions#numberofmessages) - 检索一批消息（最多 32 条）。
* [options.visibilityTimeout](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queuereceivemessageoptions#visibilitytimeout) - 设置更长或更短的不可见性超时。

以下示例使用 **receiveMessages** 方法在一次调用中获取 5 条消息。 然后，使用 `for` 循环处理每条消息。 它还会将通过此方法返回的所有消息的不可见性超时设置为 5 分钟。

```javascript
// Get up to 5 messages from the queue
const receivedMsgsResp = await queueClient.receiveMessages({ numberOfMessages: 5, visibilityTimeout: 5 * 60 });

for (i = 0; i < receivedMsgsResp.receivedMessageItems.length; i++)
{
    message = receivedMsgsResp.receivedMessageItems[i];
    console.log("Dequeuing message: ", message.messageText);
    await queueClient.deleteMessage(message.messageId, message.popReceipt);
}
```

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

可以通过两种方式自定义队列中的消息检索：

* `options.numOfMessages` - 获取一批消息（最多 32 条）。
* `options.visibilityTimeout` - 设置较长或较短的不可见性超时。

以下示例使用 **getMessages** 方法通过一次调用获取 15 条消息。 然后，使用 `for` 循环处理每条消息。 它还会将通过此方法返回的所有消息的不可见性超时设置为 5 分钟。

```javascript
queueSvc.getMessages('myqueue', {numOfMessages: 15, visibilityTimeout: 5 * 60}, function(error, results, getResponse){
  if(!error){
    // Messages retrieved
    for(var index in results){
      // text is available in result[index].messageText
      var message = results[index];
      queueSvc.deleteMessage(queueName, message.messageId, message.popReceipt, function(error, deleteResponse){
        if(!error){
          // Message deleted
        }
      });
    }
  }
});
```

---

## <a name="how-to-get-the-queue-length"></a>如何获取队列长度

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

[getProperties](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queueclient#getproperties-queuegetpropertiesoptions-) 方法返回有关队列的元数据，其中包括在队列中等待的消息的大致数目。

```javascript
const properties = await queueClient.getProperties();
console.log("Approximate queue length: ", properties.approximateMessagesCount);
```

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

**getQueueMetadata** 方法返回有关队列的元数据，其中包括在队列中等待的消息的大致数目。

```javascript
queueSvc.getQueueMetadata('myqueue', function(error, results, response){
  if(!error){
    // Queue length is available in results.approximateMessageCount
  }
});
```

---

## <a name="how-to-list-queues"></a>如何列出队列

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

若要检索队列列表，请调用 [QueueServiceClient.listQueues]()。 若要检索按特定前缀筛选的列表，请在调用 **listQueues** 时设置 [options.prefix](https://docs.microsoft.com/javascript/api/@azure/storage-queue/servicelistqueuesoptions#prefix)。

```javascript
for await (const item of queueServiceClient.listQueues()) {
  console.log("Queue: ", item.name);
}
```

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

若要检索队列的列表，请使用 **listQueuesSegmented**。 若要检索按特定前缀筛选的列表，请使用 **listQueuesSegmentedWithPrefix**。

```javascript
queueSvc.listQueuesSegmented(null, function(error, results, response){
  if(!error){
    // results.entries contains the list of queues
  }
});
```

如果无法返回所有队列，请传递 `result.continuationToken` 作为 **listQueuesSegmented** 的第一个参数或作为 **listQueuesSegmentedWithPrefix** 的第二个参数，以检索更多结果。

---

## <a name="how-to-delete-a-queue"></a>如何删除队列

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

若要删除队列以及其中包含的所有消息，请在 **QueueClient** 对象上调用 [deleteQueue](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queueclient#delete-queuedeleteoptions-) 方法。

```javascript
// Delete the queue
console.log("Deleting queue: ", queueClient.name);
await queueClient.delete();
```

若要清除队列中的所有消息而不删除该队列，请调用 [clearMessages](https://docs.microsoft.com/javascript/api/@azure/storage-queue/queueclient#clearmessages-queueclearmessagesoptions-)。

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

若要删除队列及其中包含的所有消息，请对队列对象调用 **deleteQueue** 方法。

```javascript
queueSvc.deleteQueue(queueName, function(error, response){
  if(!error){
    // Queue has been deleted
  }
});
```

若要清除队列中的所有消息而不删除该队列，请调用 **clearMessages**。

---

[!INCLUDE [storage-check-out-samples-all](../../../includes/storage-check-out-samples-all.md)]

## <a name="next-steps"></a>后续步骤

现在，了解了有关队列存储的基础知识，可单击下面的链接来了解更复杂的存储任务。

* 若要了解新增功能，请访问 [Azure 存储团队博客][Azure Storage Team Blog]
* 访问 GitHub 上的[用于 JavaScript 的 Azure 存储客户端库][Azure Storage client library for JavaScript]存储库

[Azure Storage client library for JavaScript]: https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage#azure-storage-client-library-for-javascript
[Azure Storage Team Blog]: https://techcommunity.microsoft.com/t5/azure-storage/bg-p/AzureStorageBlog
[Build and deploy a Node.js application to an Azure Cloud Service]: ../../cloud-services/cloud-services-nodejs-develop-deploy-app.md
[Create a Node.js web app in Azure App Service]: ../../app-service/app-service-web-get-started-nodejs.md
[Visual Studio Code]: https://code.visualstudio.com/docs/nodejs/nodejs-tutorial
