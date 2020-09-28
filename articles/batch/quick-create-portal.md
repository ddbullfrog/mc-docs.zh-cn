---
title: Azure 快速入门 - 在 Azure 门户中运行第一个 Batch 作业
description: 了解如何使用 Azure 门户创建 Batch 帐户、计算节点池以及在池中运行基本任务的作业。
ms.topic: quickstart
ms.service: batch
origin.date: 08/17/2020
author: rockboyfor
ms.date: 09/21/2020
ms.testscope: no
ms.testdate: 04/29/2020
ms.author: v-yeche
ms.custom: mvc
ms.openlocfilehash: 8d3181a6fb7202873dac15206d3f3a3b3af53e0b
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146263"
---
# <a name="quickstart-run-your-first-batch-job-in-the-azure-portal"></a>快速入门：在 Azure 门户中运行第一个 Batch 作业

通过使用 Azure 门户创建 Batch 帐户、计算节点（虚拟机）池以及在池中运行任务的作业，开始使用 Azure Batch。 完成本快速入门以后，你会了解 Batch 服务的重要概念，并可使用更逼真的工作负载进行更大规模的 Batch 试用。

## <a name="prerequisites"></a>先决条件

- 具有活动订阅的 Azure 帐户。 [免费创建帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="create-a-batch-account"></a>创建批处理帐户

请按以下步骤创建一个用于测试的示例 Batch 帐户。 创建池和作业需要 Batch 帐户。 如此处所示，可以将 Azure 存储帐户与 Batch 帐户相关联。 虽然本快速入门不需要，但存储帐户可以用于为大多数现实世界的工作负荷部署应用程序和存储输入和输出数据。

<!--MOONCAKE CUSOTMIZE: CORRECT ON **Virtual Machines**->

1. In the Azure portal, Select **Create a resource** > **Virtual Machines** > **Batch Service**. 

   :::image type="content" source="media/quick-create-portal/marketplace-batch.png" alt-text="Screenshot of Batch Service in the Azure Marketplace.":::

1. In the **Resource group** field, select **Create new** and enter a name for your resource group.

1. Enter a value for **Account name**. This name must be unique within the Azure **Location** selected. It can contain only lowercase letters and numbers, and it must be between 3-24 characters.

1. Under **Storage account**, select an existing storage account or create a new one.

1. Do not change any other settings. Select **Review + create**, then select **Create** to create the Batch account.

When the **Deployment succeeded** message appears, go to the Batch account that you created.

## Create a pool of compute nodes

Now that you have a Batch account, create a sample pool of Windows compute nodes for test purposes. The pool for this quick example consists of two nodes running a Windows Server 2019 image from the Azure Marketplace.

1. In the Batch account, select **Pools** > **Add**.

1. Enter a **Pool ID** called *mypool*.

1. In **Operating System**, select the following settings (you can explore other options).

    |Setting  |Value  |
    |---------|---------|
    |**Image Type**|Marketplace|
    |**Publisher** |microsoftwindowsserver|
    |**Offer** |windowsserver|
    |**Sku** |2019-datacenter-core-smalldisk|

1. Scroll down to enter **Node Size** and **Scale** settings. The suggested node size offers a good balance of performance versus cost for this quick example.

    |Setting  |Value  |
    |---------|---------|
    |**Node pricing tier** |Standard A1|
    |**Target dedicated nodes** |2|

1. Keep the defaults for remaining settings, and select **OK** to create the pool.

Batch creates the pool immediately, but it takes a few minutes to allocate and start the compute nodes. During this time, the pool's **Allocation state** is **Resizing**. You can go ahead and create a job and tasks while the pool is resizing.

After a few minutes, the allocation state changes to **Steady**, and the nodes start. To check the state of the nodes, select the pool and then select **Nodes**. When a node's state is **Idle**, it is ready to run tasks.

## Create a job

Now that you have a pool, create a job to run on it. A Batch job is a logical group for one or more tasks. A job includes settings common to the tasks, such as priority and the pool to run tasks on. Initially the job has no tasks.

1. In the Batch account view, select **Jobs** > **Add**.

1. Enter a **Job ID** called *myjob*. In **Pool**, select *mypool*. Keep the defaults for the remaining settings, and select **OK**.

## Create tasks

Now, select the job to open the **Tasks** page. This is where you'll create sample tasks to run in the job. Typically, you create multiple tasks that Batch queues and distributes to run on the compute nodes. In this example, you create two identical tasks. Each task runs a command line to display the Batch environment variables on a compute node, and then waits 90 seconds.

When you use Batch, the command line is where you specify your app or script. Batch provides a number of ways to deploy apps and scripts to compute nodes.

To create the first task:

1. Select **Add**.

1. Enter a **Task ID** called *mytask*.

1. In **Command line**, enter `cmd /c "set AZ_BATCH & timeout /t 90 > NUL"`. Keep the defaults for the remaining settings, and select **Submit**.

After you create a task, Batch queues it to run on the pool. When a node is available to run it, the task runs.

To create a second task, repeat the steps above. Enter a different **Task ID**, but specify an identical command line. If the first task is still running, Batch starts the second task on the other node in the pool.

## View task output

The example tasks you created will complete in a couple of minutes. To view the output of a completed task, select the task, then select **Files on node**. Select the file `stdout.txt` to view the standard output of the task. The contents are similar to the following:

:::image type="content" source="media/quick-create-portal/task-output.png" alt-text="Screenshot of the output from a completed task.":::

The contents show the Azure Batch environment variables that are set on the node. When you create your own Batch jobs and tasks, you can reference these environment variables in task command lines, and in the apps and scripts run by the command lines.

## Clean up resources

If you want to continue with Batch tutorials and samples, use the Batch account and linked storage account created in this quickstart. There is no charge for the Batch account itself.

You are charged for the pool while the nodes are running, even if no jobs are scheduled. When you no longer need the pool, delete it. In the account view, select **Pools** and the name of the pool. Then select **Delete**.  When you delete the pool, all task output on the nodes is deleted.

When no longer needed, delete the resource group, Batch account, and all related resources. To do so, select the resource group for the Batch account and select **Delete resource group**.

## Next steps

In this quickstart, you created a Batch account, a Batch pool, and a Batch job. The job ran sample tasks, and you viewed output created on one of the nodes. Now that you understand the key concepts of the Batch service, you are ready to try Batch with more realistic workloads at larger scale. To learn more about Azure Batch, continue to the Azure Batch tutorials.

> [!div class="nextstepaction"]
> [Azure Batch tutorials](./tutorial-parallel-dotnet.md)

<!-- Update_Description: update meta properties, wording update, update link -->