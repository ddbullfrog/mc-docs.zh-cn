---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: 启用了 GPU 的群集 - Azure Databricks
description: 了解启用了 GPU 的 Azure Databricks 群集、何时使用它们、它们的要求以及如何创建它们。
ms.openlocfilehash: 1111a14d539ce3c6fe550893f453c27c803641af
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121797"
---
# <a name="gpu-enabled-clusters"></a><a id="gpu-clusters"> </a><a id="gpu-enabled-clusters"> </a>启用了 GPU 的群集

> [!NOTE]
>
> 某些启用了 GPU 的实例类型为 **Beta 版本** 。当你在群集创建过程中选择驱动程序和工作器类型时，这些类型将在下拉列表中如此标记。

## <a name="overview"></a>概述

Azure Databricks 支持通过图形处理单元 (GPU) 加速的群集。
本文介绍了如何创建包含启用了 GPU 的实例的群集，并介绍了在这些实例上安装的 GPU 驱动程序和库。

若要详细了解启用了 GPU 的群集上的深度学习，请参阅[深度学习](../applications/machine-learning/train-model/deep-learning.md)。

## <a name="create-a-gpu-cluster"></a>创建 GPU 群集

创建 GPU 群集类似于创建任何 Spark 群集（请参阅[群集](index.md)）。
应当记住以下事项：

* **Databricks Runtime 版本** 必须是启用了 GPU 的版本，例如 **Runtime 6.6 ML（GPU、Scala 2.11、Spark 2.4.5）** 。
* **工作器类型** 和 **驱动程序类型** 必须为 GPU 实例类型。
* 对于没有 Spark 的单计算机工作流，可以将工作器数目设置为零。

Azure Databricks 支持 NC 实例类型系列： **NC12** 和 **NC24** 以及 NCv3 实例类型系列： **NC6s_v3** 、 **NC12s_v3** 和 **NC24s_v3** 。 有关支持的 GPU 实例类型及其可用性区域的最新列表，请参阅 [Azure Databricks 定价](https://www.azure.cn/pricing/details/databricks/#instances)。
你的 Azure Databricks 部署必须位于受支持的区域内，才能启动启用了 GPU 的群集。

## <a name="gpu-scheduling"></a>GPU 调度

Databricks Runtime 7.0 ML 及更高版本支持 Apache Spark 3.0 中提供的 [GPU 感知型计划](https://spark.apache.org/docs/3.0.0-preview/configuration.html#custom-resource-scheduling-and-configuration-overview)。 Azure Databricks 在 GPU 群集上为你预配置该计划。

`spark.task.resource.gpu.amount` 是你可能需要更改的与 GPU 感知型计划相关的唯一 Spark 配置。
默认配置为每个任务使用一个 GPU。如果你使用所有 GPU 节点，这对于分布式推理工作负荷和分布式训练非常理想。
如果你想要在部分节点上进行分布式训练（这有助于减少分布式训练过程中的通信开销），Databricks 建议在群集 [Spark 配置](configure.md#spark-config)中将 `spark.task.resource.gpu.amount` 设置为每个工作器节点的 GPU 数。

对于 PySpark 任务，Azure Databricks 自动将分配的 GPU 重新映射到索引 0、1…。
在为每个任务使用一个 GPU 的默认配置下，你的代码可以直接使用默认 GPU，不需检查为任务分配了哪个 GPU。
如果为每个任务设置了多个 GPU（例如 4 个），则你的代码可以假设已分配 GPU 的索引始终为 0、1、2、3。 如果你需要已分配 GPU 的物理索引，则可从 `CUDA_VISIBLE_DEVICES` 环境变量获取它们。

如果你使用 Scala，则可从 `TaskContext.resources().get("gpu")` 获取分配给任务的 GPU 的索引。

对于低于 7.0 的 Databricks Runtime 版本，为了避免多个尝试使用同一 GPU 的 Spark 任务之间发生冲突，Azure Databricks 会自动配置 GPU 群集，使每个节点最多有一个正在运行的任务。
在这种情况下，任务可以使用节点上的所有 GPU，而不会与其他任务发生冲突。

## <a name="nvidia-gpu-driver-cuda-and-cudnn"></a><a id="nvidia"> </a><a id="nvidia-gpu-driver-cuda-and-cudnn"> </a>NVIDIA GPU 驱动程序、CUDA 和 cuDNN

Azure Databricks 在 Spark 驱动程序和工作器实例上安装使用 GPU 所需的 NVIDIA 驱动程序和库：

* [CUDA 工具包](https://developer.nvidia.com/cuda-toolkit)，安装在 `/usr/local/cuda`下。
* [cuDNN](https://developer.nvidia.com/cudnn)：NVIDIA CUDA 深度神经网络库。
* [NCCL](https://developer.nvidia.com/nccl)：NVIDIA 集体通信库。

包含的 NVIDIA 驱动程序的版本为 440.64。
有关所包含的库的版本，请参阅你使用的特定 Databricks Runtime 版本的[发行说明](../release-notes/runtime/index.md#runtime-release-notes)。

> [!NOTE]
>
> 此软件包含 NVIDIA Corporation 提供的源代码。 具体而言，为支持 GPU，Azure Databricks 包括了 [CUDA 示例](https://docs.nvidia.com/cuda/eula/#nvidia-cuda-samples-preface)中的代码。

### <a name="nvidia-end-user-license-agreement-eula"></a>NVIDIA 最终用户许可协议 (EULA)

当你在 Azure Databricks 中选择启用了 GPU 的“Databricks Runtime 版本”时，你默示同意 [NVIDIA EULA](../_static/documents/nvidia-cloud-end-user-license-agreement_clean.pdf) 中列出的有关 CUDA、cuDNN 和 Tesla 库的条款和条件，以及 NCCL 库的 [NVIDIA 最终用户许可协议（包含 NCCL 补充条款）](https://docs.nvidia.com/deeplearning/sdk/nccl-sla/index.html#supplement/)。

## <a name="databricks-container-services-on-gpu-clusters"></a>GPU 群集上的 Databricks 容器服务

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../release-notes/release-types.md)提供。

[Databricks 容器服务](custom-containers.md)可在具有 GPU 的群集上用来通过自定义库创建可移植的深度学习环境。 有关说明，请参阅 [Databricks 容器服务的文档](custom-containers.md)。

Databricks Runtime [Docker Hub](https://hub.docker.com/u/databricksruntime) 包含具有 GPU 功能的示例基础映像。 用于生成这些映像的 Dockerfile 位于[示例容器 GitHub 存储库](https://github.com/databricks/containers/tree/master/ubuntu/gpu)，其中还详细介绍了示例图像提供的内容以及如何对其进行自定义。

为 GPU 群集创建自定义映像时，无法更改 NVIDIA 驱动程序版本。 NVIDIA 驱动程序版本必须与主计算机上的驱动程序版本 440.64 相匹配。 此版本不支持 CUDA 11。