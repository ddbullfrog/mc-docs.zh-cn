---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/24/2020
title: 群集 - Azure Databricks
description: 了解 Azure Databricks 群集及其创建和管理方式。
ms.openlocfilehash: b5ce417de2c50a70b6af79277d60efb2f379b61d
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121796"
---
# <a name="clusters"></a>群集

Azure Databricks 群集是一组计算资源和配置，在其中可以运行数据工程、数据科学和数据分析工作负荷，例如生产 ETL 管道、流分析、即席分析和机器学习。

可将这些工作负荷作为[笔记本](../notebooks/index.md)中的一组命令运行，或者作为自动化[作业](../jobs.md)运行。 Azure Databricks 会区分通用群集和作业群集 。 借助通用群集，可通过交互式笔记本以协作的方式分析数据。 借助作业群集，可运行快速可靠的自动化作业。

* 可使用 UI、CLI 或 REST API 创建通用群集。 可手动终止和重启通用群集。 多个用户可以共享此类群集，以协作的方式执行交互式分析。
* 当你在新的作业群集上运行[作业](../jobs.md)时，Azure Databricks 作业计划程序将创建一个作业群集，并在作业完成时终止该群集 。 无法重启作业群集。

此部分介绍如何通过 UI 来使用群集。 有关其他方法，请参阅[群集 CLI](../dev-tools/cli/clusters-cli.md) 和[群集 API](../dev-tools/api/latest/clusters.md)。

此外，本部分将重点放在通用群集而不是作业群集上，不过，所述的许多配置和管理工具对于这两种群集类型同样适用。 若要详细了解如何创建作业群集，请参阅[作业](../jobs.md)。

> [!IMPORTANT]
>
> Azure Databricks 保留最近 30 天内终止的最多 70 个通用群集的群集配置信息，以及作业计划程序最近终止的最多 30 个作业群集的群集配置信息。 若要在通用群集已[终止](clusters-manage.md#cluster-terminate)超过 30 天后仍保留通用群集配置，管理员可将群集[固定](clusters-manage.md#cluster-pin)到群集列表。

本部分内容：

* [创建群集](create.md)
* [管理群集](clusters-manage.md)
  * [显示分类](clusters-manage.md#display-clusters)
  * [固定群集](clusters-manage.md#pin-a-cluster)
  * [以 JSON 文件的形式查看群集配置](clusters-manage.md#view-a-cluster-configuration-as-a-json-file)
  * [编辑群集](clusters-manage.md#edit-a-cluster)
  * [克隆群集](clusters-manage.md#clone-a-cluster)
  * [控制对群集的访问](clusters-manage.md#control-access-to-clusters)
  * [启动群集](clusters-manage.md#start-a-cluster)
  * [终止群集](clusters-manage.md#terminate-a-cluster)
  * [删除群集](clusters-manage.md#delete-a-cluster)
  * [在 Apache Spark UI 中查看群集信息](clusters-manage.md#view-cluster-information-in-the-apache-spark-ui)
  * [查看群集日志](clusters-manage.md#view-cluster-logs)
  * [监视性能](clusters-manage.md#monitor-performance)
* [配置群集](configure.md)
  * [群集策略](configure.md#cluster-policy)
  * [群集模式](configure.md#cluster-mode)
  * [池](configure.md#pool)
  * [Databricks Runtime](configure.md#databricks-runtime)
  * [Python 版本](configure.md#python-version)
  * [群集节点类型](configure.md#cluster-node-type)
  * [群集大小和自动缩放](configure.md#cluster-size-and-autoscaling)
  * [自动缩放本地存储](configure.md#autoscaling-local-storage)
  * [Spark 配置](configure.md#spark-configuration)
  * [启用本地磁盘加密](configure.md#enable-local-disk-encryption)
  * [环境变量](configure.md#environment-variables)
  * [群集标记](configure.md#cluster-tags)
  * [通过 SSH 访问群集](configure.md#ssh-access-to-clusters)
  * [群集日志传送](configure.md#cluster-log-delivery)
  * [初始化脚本](configure.md#init-scripts)
* [任务抢占](preemption.md)
  * [抢占选项](preemption.md#preemption-options)
* [使用 Databricks 容器服务自定义容器](custom-containers.md)
  * [要求](custom-containers.md#requirements)
  * [步骤 1：生成基础映像](custom-containers.md#step-1-build-your-base)
  * [步骤 2：推送基础映像](custom-containers.md#step-2-push-your-base-image)
  * [步骤 3：启动群集](custom-containers.md#step-3-launch-your-cluster)
  * [使用初始化脚本](custom-containers.md#use-an-init-script)
* [群集节点初始化脚本](init-scripts.md)
  * [初始化脚本类型](init-scripts.md#init-script-types)
  * [初始化脚本执行顺序](init-scripts.md#init-script-execution-order)
  * [环境变量](init-scripts.md#environment-variables)
  * [Logging](init-scripts.md#logging)
  * [群集范围的初始化脚本](init-scripts.md#cluster-scoped-init-scripts)
  * [全局初始化脚本（新）](init-scripts.md#global-init-scripts-new)
  * [旧版全局初始化脚本（已弃用）](init-scripts.md#legacy-global-init-scripts-deprecated)
  * [群集命名的初始化脚本（已弃用）](init-scripts.md#cluster-named-init-scripts-deprecated)
  * [旧版全局和群集命名的初始化脚本日志（已弃用）](init-scripts.md#legacy-global-and-cluster-named-init-script-logs-deprecated)
* [支持 GPU 的群集](gpu.md)
  * [概述](gpu.md#overview)
  * [创建 GPU 群集](gpu.md#create-a-gpu-cluster)
  * [GPU 调度](gpu.md#gpu-scheduling)
  * [NVIDIA GPU 驱动程序、CUDA 和 cuDNN](gpu.md#nvidia-gpu-driver-cuda-and-cudnn)
  * [GPU 群集上的 Databricks 容器服务](gpu.md#databricks-container-services-on-gpu-clusters)
* [单节点群集](single-node.md)
  * [创建单节点群集](single-node.md#create-a-single-node-cluster)
  * [单节点群集属性](single-node.md#single-node-cluster-properties)
  * [限制](single-node.md#limitations)
  * [单节点群集策略](single-node.md#single-node-cluster-policy)
  * [单节点作业群集策略](single-node.md#single-node-job-cluster-policy)
* [池](instance-pools/index.md)
  * [显示池](instance-pools/display.md)
  * [创建池](instance-pools/create.md)
  * [池配置](instance-pools/configure.md)
  * [编辑池](instance-pools/edit.md)
  * [删除池](instance-pools/delete.md)
  * [使用池](instance-pools/cluster-instance-pool.md)
* [Web 终端](web-terminal.md)
  * [要求](web-terminal.md#requirements)
  * [使用 Web 终端](web-terminal.md#use-the-web-terminal)
  * [限制](web-terminal.md#limitations)