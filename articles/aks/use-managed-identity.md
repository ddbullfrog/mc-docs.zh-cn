---
title: 在 Azure Kubernetes 服务中使用托管标识
description: 了解如何在 Azure Kubernetes 服务 (AKS) 中使用托管标识
services: container-service
ms.topic: article
author: rockboyfor
ms.date: 10/12/2020
ms.testscope: no
ms.testdate: 07/13/2020
ms.author: v-yeche
ms.openlocfilehash: e603b0c1a6f572c1bca09c18c927a9223d29bb74
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128264"
---
<!--Verified successfully-->
# <a name="use-managed-identities-in-azure-kubernetes-service"></a>在 Azure Kubernetes 服务中使用托管标识

目前，Azure Kubernetes 服务 (AKS) 群集（特指 Kubernetes 云提供商）需要使用标识才能在 Azure 中创建其他资源，例如负载均衡器和托管磁盘。 此标识可以是托管标识或服务主体。 如果使用[服务主体](kubernetes-service-principal.md)，你必须提供一个服务主体，或由 AKS 代表你创建一个。 如果使用托管标识，AKS 会自动为你创建托管标识。 使用服务主体的群集最终会达到这样一种状态，即，必须续订服务主体才能让群集保持正常运行。 管理服务主体会增加复杂性，这也是托管标识使用起来更简单的原因。 服务主体和托管标识适用相同的权限要求。

托管标识本质上是服务主体的包装器，这使其更易于管理。 根据 Azure Active Directory 的默认设置，MI 的凭据轮换每 46 天自动发生一次。 AKS 使用系统分配和用户分配的托管标识类型。 这些标识目前是不可变的。 若要了解详细信息，请阅读 [Azure 资源托管标识](../active-directory/managed-identities-azure-resources/overview.md)。

## <a name="before-you-begin"></a>准备阶段

必须安装了以下资源：

- Azure CLI 版本 2.8.0 或更高版本

## <a name="limitations"></a>限制

* 具有托管标识的 AKS 群集只能在群集创建过程中启用。
* 现有 AKS 群集无法迁移到托管标识。
* 在群集升级操作期间，托管标识暂时不可用。
* 不支持启用了托管标识的群集的租户移动/迁移。
* 如果群集启用了 `aad-pod-identity`，节点托管标识 (NMI) pod 将修改节点的 iptable，以拦截对 Azure 实例元数据终结点的调用。 此配置意味着对元数据终结点发出的任何请求都将被 NMI 拦截，即使 pod 不使用 `aad-pod-identity`。 可以将 AzurePodIdentityException CRD 配置为通知 `aad-pod-identity` 应在不使用 NMI 进行出任何处理的情况下，代理与 CRD 中定义的标签匹配的 pod 所发起的对元数据终结点的任何请求。 应通过配置 AzurePodIdentityException CRD 在 `aad-pod-identity` 中排除在 _kube-system_ 命名空间中具有 `kubernetes.azure.com/managedby: aks` 标签的系统 pod。 有关详细信息，请参阅[禁用特定 pod 或应用程序的 aad-pod-identity](https://github.com/Azure/aad-pod-identity/blob/master/docs/readmes/README.app-exception.md)。
    若要配置例外情况，请安装 [mic-exception YAML](https://github.com/Azure/aad-pod-identity/blob/master/deploy/infra/mic-exception.yaml)。

## <a name="summary-of-managed-identities"></a>托管标识摘要

AKS 对内置服务和加载项使用多个托管标识。

<!--MOONCAKE: REMOVE `Bring your own identity` COLUMN DETAILS-->

| 标识                       | 名称    | 使用案例 | 默认权限 |
|----------------------------|-----------|----------|
| 控制面板 | 不可见 | 由 AKS 用于托管网络资源，包括入口负载均衡器和 AKS 托管公共 IP | 节点资源组的参与者角色 |
| Kubelet | AKS Cluster Name-agentpool | 向 Azure 容器注册表 (ACR) 进行身份验证 | NA（对于 kubernetes v1.15+） |
| 加载项 | AzureNPM | 无需标识 | 不可用 |
| 加载项 | AzureCNI 网络监视 | 无需标识 | 不可用 |
| 加载项 | azurepolicy（网关守卫） | 无需标识 | 不可用 |
| 加载项 | azurepolicy | 无需标识 | 不可用 |
| 加载项 | Calico | 无需标识 | 不可用 |
| 加载项 | 仪表板 | 无需标识 | 不可用 |
| 加载项 | HTTPApplicationRouting | 管理所需的网络资源 | 节点资源组的读取者角色，DNS 区域的参与者角色 |
| 加载项 | 入口应用程序网关 | 管理所需的网络资源| 节点资源组的参与者角色 |
| 加载项 | omsagent | 用于将 AKS 指标发送到 Azure Monitor | “监视指标发布者”角色 |
| 加载项 | Virtual-Node (ACIConnector) | 管理 Azure 容器实例 (ACI) 所需的网络资源 | 节点资源组的参与者角色 |
| OSS 项目 | aad-pod-identity | 通过 Azure Active Directory (AAD) 使应用程序可安全访问云资源 | NA |

## <a name="create-an-aks-cluster-with-managed-identities"></a>创建具有托管标识的 AKS 群集

现在，可以使用以下 CLI 命令创建具有托管标识的 AKS 群集。

首先，创建 Azure 资源组：

```azurecli
# Create an Azure resource group
az group create --name myResourceGroup --location chinaeast2
```

然后，创建 AKS 群集：

```azurecli
az aks create -g myResourceGroup -n myManagedCluster --enable-managed-identity
```

使用托管标识成功创建群集的命令中包含以下服务主体配置文件信息：

```output
"servicePrincipalProfile": {
    "clientId": "msi"
  }
```

使用以下命令查询控制平面托管标识的 objectid：

```azurecli
az aks show -g myResourceGroup -n myManagedCluster --query "identity"
```

结果应如下所示：

```output
{
  "principalId": "<object_id>",   
  "tenantId": "<tenant_id>",      
  "type": "SystemAssigned"                                 
}
```

创建群集后，你便可以将应用程序工作负荷部署到新群集中，并与之交互，就像与基于服务主体的 AKS 群集交互一样。

> [!NOTE]
> 若要创建并使用自己的 VNet、静态 IP 地址或附加的 Azure 磁盘（资源位于工作器节点资源组外部），请使用群集系统分配的托管标识的 PrincipalID 来执行角色分配。 有关角色分配的详细信息，请参阅[委托对其他 Azure 资源的访问权限](kubernetes-service-principal.md#delegate-access-to-other-azure-resources)。
>
> 向 Azure 云提供商使用的群集托管标识授予的权限可能需要 60 分钟才能填充完毕。

最后，获取用于访问群集的凭据：

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myManagedCluster
```

<!--Not Available on ## Bring your own control plane MI (Preview)-->
<!--Not Available on ## Next steps-->
<!--Not Available on [Azure Resource Manager (ARM) templates ][aks-arm-template]-->

<!-- LINKS - external -->

<!--Not Available on [aks-arm-template]: https://docs.microsoft.com/azure/templates/microsoft.containerservice/managedclusters-->

[az-identity-create]: https://docs.microsoft.com/cli/azure/identity?view=azure-cli-latest#az_identity_create&preserve-view=true
[az-identity-list]: https://docs.microsoft.com/cli/azure/identity?view=azure-cli-latest#az_identity_list&preserve-view=true

<!-- Update_Description: update meta properties, wording update, update link -->
