---
title: 创建并附加 Azure Kubernetes 服务
titleSuffix: Azure Machine Learning
description: Azure Kubernetes 服务 (AKS) 可用于将机器学习模型部署为 Web 服务。 了解如何通过 Azure 机器学习创建新的 AKS 群集。 你还将了解如何将现有的 AKS 群集附加到 Azure 机器学习工作区。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 09/01/2020
ms.openlocfilehash: 4eea2635c390444a59314c0dff62495bcfe20831
ms.sourcegitcommit: 71953ae66ddfc07c5d3b4eb55ff8639281f39b40
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2020
ms.locfileid: "91395578"
---
# <a name="create-and-attach-an-azure-kubernetes-service-cluster"></a>创建并附加 Azure Kubernetes 服务群集
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Azure 机器学习可以将经过训练的机器学习模型部署到 Azure Kubernetes 服务。 但是，必须首先从 Azure ML 工作区创建 Azure Kubernetes 服务 (AKS) 群集，或者附加现有 AKS 群集。 本文提供了有关创建和附加群集的信息。

## <a name="prerequisites"></a>先决条件

- Azure 机器学习工作区。 有关详细信息，请参阅[创建 Azure 机器学习工作区](how-to-manage-workspace.md)。

- [机器学习服务的 Azure CLI 扩展](reference-azure-machine-learning-cli.md)、[Azure 机器学习 Python SDK](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true) 或 [Azure 机器学习 Visual Studio Code 扩展](tutorial-setup-vscode-extension.md)。

- 如果计划使用 Azure 虚拟网络来保护 Azure ML 工作区与 AKS 群集之间的通信，请阅读[训练和推理期间的网络隔离](how-to-enable-virtual-network.md)一文。

## <a name="limitations"></a>限制

- 如果群集中需要部署的是标准负载均衡器 (SLB)，而不是基本负载均衡器 (BLB)，请在 AKS 门户/CLI/SDK 中创建群集，然后将该群集附加到 AML 工作区 。

- 如果你的 Azure Policy 限制创建公共 IP 地址，则无法创建 AKS 群集。 AKS 需要一个公共 IP 用于[出口流量](/aks/limit-egress-traffic)。 出口流量一文还指导如何通过公共 IP（几个完全限定域名的 IP 除外）锁定来自群集的出口流量。 启用公共 IP 有两种方法：
    - 群集可以使用在默认情况下与 BLB 或 SLB 一起创建的公共 IP，或者
    - 可以在没有公共 IP 的情况下创建群集，然后为公共 IP 配置一个带有用户定义路由的防火墙。 有关详细信息，请参阅[使用用户定义的路由自定义群集出口](/aks/egress-outboundtype)。
    
    AML 控制平面不会与此公共 IP 通信。 它与 AKS 控制平面通信以便进行部署。 

- 如果附加 AKS 群集（已[启用授权 IP 范围来访问 API 服务器](/aks/api-server-authorized-ip-ranges)），请为该 AKS 群集启用 AML 控制平面 IP 范围。 AML 控制平面是跨配对区域部署的，并且会在 AKS 群集上部署推理 Pod。 如果无法访问 API 服务器，则无法部署推理 Pod。 在 AKS 群集中启用 IP 范围时，请对两个[配对区域](/best-practices-availability-paired-regions)都使用 [IP 范围](https://www.microsoft.com/download/confirmation.aspx?id=56519)。

    授权 IP 范围仅适用于标准负载均衡器。


- AKS 群集的计算名称在 Azure ML 工作区中必须是唯一的。
    - 名称是必须提供的，且长度必须介于 3 到 24 个字符之间。
    - 有效字符为大小写字母、数字和 - 字符。
    - 名称必须以字母开头。
    - 名称必须在 Azure 区域内的全部现有计算中都是唯一的。 如果选择的名称不是唯一的，则会显示警报。
   
 - 如果要将模型部署到 GPU 节点或 FPGA 节点（或任何特定 SKU），则必须使用该特定 SKU 创建群集。 不支持在现有群集中创建辅助节点池以及在辅助节点池中部署模型。
 
- 创建或附加群集时，可以选择为开发/测试还是生产创建群集。 如果要创建 AKS 群集以用于开发、验证和测试而非生产，请将“群集用途”设置为“开发/测试”    。 如果未指定群集用途，则会创建生产群集。 

    > [!IMPORTANT]
    > 开发/测试群集不适用于生产级别的流量，并且可能会增加推理时间。 开发/测试群集也不保证容错能力。

- 创建或附加群集时，如果该群集将用于生产，则它必须包含至少 12 个虚拟 CPU。 虚拟 CPU 数量的计算公式为群集中的节点数乘以所选 VM 大小提供的核心数。 例如，如果使用的 VM 大小为“Standard_D3_v2”（具有 4 个虚拟核心），则应该为节点数选择 3 个或更大的数字。

    对于开发/测试群集，建议至少拥有 2 个虚拟 CPU。

- Azure 机器学习 SDK 不支持缩放 AKS 群集。 要缩放群集中的节点，请在 Azure 机器学习工作室中使用 AKS 群集的 UI。 只能更改节点计数，不能更改群集的 VM 大小。 有关缩放 AKS 群集中节点的详细信息，请参阅以下文章：

    - [手动缩放 AKS 群集中的节点计数](../aks/scale-cluster.md)
    - [在 AKS 中设置群集自动缩放程序](../aks/cluster-autoscaler.md)

## <a name="create-a-new-aks-cluster"></a>创建新的 AKS 群集

**时间估计**：大约 10 分钟。

对于工作区而言，创建或附加 AKS 群集是一次性过程。 可以将此群集重复用于多个部署。 如果删除该群集或包含该群集的资源组，则在下次需要进行部署时必须创建新群集。 可将多个 AKS 群集附加到工作区。

以下示例演示如何使用 SDK 和 CLI 创建新的 AKS 群集：

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.compute import AksCompute, ComputeTarget

# Use the default configuration (you can also provide parameters to customize this).
# For example, to create a dev/test cluster, use:
# prov_config = AksCompute.provisioning_configuration(cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST)
prov_config = AksCompute.provisioning_configuration()

# Example configuration to use an existing virtual network
# prov_config.vnet_name = "mynetwork"
# prov_config.vnet_resourcegroup_name = "mygroup"
# prov_config.subnet_name = "default"
# prov_config.service_cidr = "10.0.0.0/16"
# prov_config.dns_service_ip = "10.0.0.10"
# prov_config.docker_bridge_cidr = "172.17.0.1/16"

aks_name = 'myaks'
# Create the cluster
aks_target = ComputeTarget.create(workspace = ws,
                                    name = aks_name,
                                    provisioning_configuration = prov_config)

# Wait for the create process to complete
aks_target.wait_for_completion(show_output = True)
```

有关此示例中使用的类、方法和参数的详细信息，请参阅以下参考文档：

* [AksCompute.ClusterPurpose](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.akscompute.clusterpurpose?view=azure-ml-py&preserve-view=true)
* [AksCompute.provisioning_configuration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)
* [ComputeTarget.create](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computetarget?view=azure-ml-py#create-workspace--name--provisioning-configuration-)
* [ComputeTarget.wait_for_completion](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computetarget?view=azure-ml-py#wait-for-completion-show-output-false-)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az ml computetarget create aks -n myaks
```

有关详细信息，请参阅 [az ml computetarget create aks](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/create?view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-create-aks) 参考文档。

# <a name="portal"></a>[门户](#tab/azure-portal)

有关在门户中创建 AKS 群集的信息，请参阅[在 Azure 机器学习工作室中创建计算目标](how-to-create-attach-compute-studio.md#inference-clusters)。

---

## <a name="attach-an-existing-aks-cluster"></a>附加现有的 AKS 群集

时间估计****：大约 5 分钟。

如果 Azure 订阅中已有 AKS 群集并且其版本为 1.17 或更低版本，则可以使用该群集来部署映像。

> [!TIP]
> 现有的 AKS 群集除了位于 Azure 机器学习工作区，还可位于 Azure 区域中。


> [!WARNING]
> 请勿在工作区中为同一 AKS 群集创建多个同步附件。 例如，使用两个不同的名称将一个 AKS 群集附加到工作区。 每个新附件都会破坏先前存在的附件。
>
> 如果要重新附加 AKS 群集（例如，更改 TLS 或其他群集配置设置），则必须先使用 [AksCompute.detach()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#detach--) 删除现有附件。

有关如何使用 Azure CLI 或门户创建 AKS 群集的详细信息，请参阅以下文章：

* [创建 AKS 群集 (CLI)](https://docs.microsoft.com/cli/azure/aks?toc=%2Fazure%2Faks%2FTOC.json&bc=%2Fazure%2Fbread%2Ftoc.json&view=azure-cli-latest#az-aks-create)
* [创建 AKS 群集（门户）](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough-portal?view=azure-cli-latest)
* [创建 AKS 群集（Azure 快速入门模板上的 ARM 模板）](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aks-azml-targetcompute)

以下示例演示如何将现有 AKS 群集附加到工作区：

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.compute import AksCompute, ComputeTarget
# Set the resource group that contains the AKS cluster and the cluster name
resource_group = 'myresourcegroup'
cluster_name = 'myexistingcluster'

# Attach the cluster to your workgroup. If the cluster has less than 12 virtual CPUs, use the following instead:
# attach_config = AksCompute.attach_configuration(resource_group = resource_group,
#                                         cluster_name = cluster_name,
#                                         cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST)
attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                         cluster_name = cluster_name)
aks_target = ComputeTarget.attach(ws, 'myaks', attach_config)

# Wait for the attach process to complete
aks_target.wait_for_completion(show_output = True)
```

有关此示例中使用的类、方法和参数的详细信息，请参阅以下参考文档：

* [AksCompute.attach_configuration()](/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)
* [AksCompute.ClusterPurpose](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.akscompute.clusterpurpose?view=azure-ml-py&preserve-view=true)
* [AksCompute.attach](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computetarget?view=azure-ml-py#attach-workspace--name--attach-configuration-)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

要使用 CLI 附加现有群集，需要获取现有群集的资源 ID。 请使用以下命令要获取该值。 将 `myexistingcluster` 替换为 AKS 群集的名称。 将 `myresourcegroup` 替换为包含该群集的资源组：

```azurecli
az aks show -n myexistingcluster -g myresourcegroup --query id
```

此命令返回类似于以下文本的值：

```text
/subscriptions/{GUID}/resourcegroups/{myresourcegroup}/providers/Microsoft.ContainerService/managedClusters/{myexistingcluster}
```

要将现有群集附加到工作区，请使用以下命令。 将 `aksresourceid` 替换为上一命令返回的值。 将 `myresourcegroup` 替换为包含工作区的资源组。 将 `myworkspace` 替换为工作区名称。

```azurecli
az ml computetarget attach aks -n myaks -i aksresourceid -g myresourcegroup -w myworkspace
```

有关详细信息，请参阅 [az ml computetarget attach aks](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/attach?view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-attach-aks) 参考文档。

# <a name="portal"></a>[门户](#tab/azure-portal)

有关在门户中附加 AKS 群集的信息，请参阅[在 Azure 机器学习工作室中创建计算目标](how-to-create-attach-compute-studio.md#inference-clusters)。

---

## <a name="next-steps"></a>后续步骤

* [部署模型的方式和位置](how-to-deploy-and-where.md)
* [将模型部署到 Azure Kubernetes 服务群集](how-to-deploy-azure-kubernetes-service.md)