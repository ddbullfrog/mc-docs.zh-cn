---
title: Azure 上的 Kubernetes 教程 - 部署群集
description: 此 Azure Kubernetes 服务 (AKS) 教程介绍如何创建 AKS 群集并使用 kubectl 连接到 Kubernetes 主节点。
services: container-service
ms.topic: tutorial
origin.date: 09/30/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: no
ms.testdate: 05/25/2020
ms.author: v-yeche
ms.custom: mvc
ms.openlocfilehash: f815f27b24db0caa74c25bcc2c1c35c2c0f72b57
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470188"
---
# <a name="tutorial-deploy-an-azure-kubernetes-service-aks-cluster"></a>教程：部署 Azure Kubernetes 服务 (AKS) 群集

Kubernetes 为容器化应用程序提供一个分布式平台。 使用 AKS 可以快速创建生产就绪的 Kubernetes 群集。 在本教程的第 3 部分（共 7 部分）中，在 AKS 中部署了 Kubernetes 群集。 学习如何：

> [!div class="checklist"]
> * 部署可对 Azure 容器注册表进行身份验证的 Kubernetes AKS 群集
> * 安装 Kubernetes CLI (kubectl)
> * 配置 kubectl，以便连接到 AKS 群集

在其他教程中，Azure 投票应用程序将部署到群集，并进行缩放和更新。

## <a name="before-you-begin"></a>开始之前

在以前的教程中，已创建容器映像并上传到 Azure 容器注册表实例。 如果尚未完成这些步骤，并且想要逐一完成，请先参阅[教程 1 - 创建容器映像][aks-tutorial-prepare-app]。

此教程需要运行 Azure CLI 2.0.53 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI][azure-cli-install]。

## <a name="create-a-kubernetes-cluster"></a>创建 Kubernetes 群集

AKS 群集可以使用 Kubernetes 基于角色的访问控制 (RBAC)。 可以使用这些控制根据分配给用户的角色定义资源访问权限。 权限可以组合（如果为用户分配了多个角色），可以局限于单个命名空间，也可以涵盖整个群集。 默认情况下，Azure CLI 会在你创建 AKS 群集时自动启用 RBAC。

使用 [az aks create][] 创建 AKS 群集。 以下示例在名为 *myResourceGroup* 的资源组中创建名为 *myAKSCluster* 的群集。 此资源组是[上一教程][aks-tutorial-prepare-acr]中在 chinaeast2 区域中创建的。 下面的示例未指定区域，因此 AKS 群集也会在 chinaeast2 区域中创建。 请参阅 [Azure Kubernetes 服务 (AKS) 中的配额、虚拟机大小限制和区域可用性][quotas-skus-regions]，以了解有关 AKS 的资源限制和区域可用性的详细信息。

为了允许 AKS 群集与其他 Azure 资源进行交互，将自动创建一个 Azure Active Directory 服务主体，因为未指定该主体。 在这里，此服务主体[被授予从上一教程中创建的 Azure 容器注册表 (ACR) 实例中拉取映像][container-registry-integration]的权限。 请注意，可以使用[托管标识](use-managed-identity.md)而不是服务主体，以便更轻松地进行管理。

```azurecli
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 2 \
    --generate-ssh-keys \
    --attach-acr <acrName>
```

还可以手动将服务主体配置为从 ACR 中拉取映像。 有关详细信息，请参阅[使用服务主体进行 ACR 身份验证](../container-registry/container-registry-auth-service-principal.md)或[使用请求密码从 Kubernetes 进行身份验证](../container-registry/container-registry-auth-kubernetes.md)。

几分钟后，部署完成并返回有关 AKS 部署的 JSON 格式信息。

> [!NOTE]
> 若要确保群集能够可靠运行，应至少运行 2（两个）节点。

## <a name="install-the-kubernetes-cli"></a>安装 Kubernetes CLI

若要从本地计算机连接到 Kubernetes 群集，请使用 [kubectl][kubectl]（Kubernetes 命令行客户端）。

<!--MOONCAKE CUSTOMIZATION CONTENT 03/28/2019-->
<!--Sync with 3. Install kubectl in https://github.com/Azure/container-service-for-azure-china/tree/master/aks--> 

如果使用 Azure 本地 Shell，也可使用 [az aks install-cli][] 命令在本地安装它：

```azurecli
az aks install-cli --install-location <kubectl-download-path>
```

> [!NOTE]
> 原始的 `az aks install-cli` 命令在 Azure 中国区无效，详见[此文](https://mirror.azk8s.cn/help/kubernetes.html)。

* 可以通过 PR [添加适用于 Azure 中国区的“az aks install-cli”支持](https://github.com/Azure/azure-cli/pull/8675)来修复此问题。以下命令会在 Linux 上启动容器化的 azure-cli(`dockerhub.azk8s.cn/andyzhangx/azure-cli:v2.0.60-china`) 将最新的 `kubectl` 版本下载到 `/usr/local/bin/`：
    
    ```
    # docker run -v ${HOME}:/root -v /usr/local/bin/:/kube -it dockerhub.azk8s.cn/andyzhangx/azure-cli:v2.0.60-china
    root@09feb993f352:/# az cloud set --name AzureChinaCloud
    root@09feb993f352:/# az aks install-cli --install-location /kube/kubectl
    ```

<!--MOONCAKE CUSTOMIZATION CONTENT 03/28/2019-->

## <a name="connect-to-cluster-using-kubectl"></a>使用 kubectl 连接到群集

若要将 `kubectl` 配置为连接到 Kubernetes 群集，请使用 [az aks get-credentials][] 命令。 以下示例获取 myResourceGroup  中名为“myAKSCluster”  的 AKS 群集的凭据：

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

若要验证与群集的连接，请运行 [kubectl get nodes][kubectl-get] 命令以返回群集节点列表：

```
$ kubectl get nodes

NAME                       STATUS   ROLES   AGE   VERSION
aks-nodepool1-12345678-0   Ready    agent   32m   v1.14.8
```

## <a name="next-steps"></a>后续步骤

本教程在 AKS 中部署了一个 Kubernetes 群集并将 `kubectl` 配置为连接到该群集。 你已了解如何执行以下操作：

> [!div class="checklist"]
> * 部署可对 Azure 容器注册表进行身份验证的 Kubernetes AKS 群集
> * 安装 Kubernetes CLI (kubectl)
> * 配置 kubectl，以便连接到 AKS 群集

请继续学习下一教程，了解如何将应用程序部署到群集。

> [!div class="nextstepaction"]
> [在 Kubernetes 中部署应用程序][aks-tutorial-deploy-app]

<!-- LINKS - external -->

[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- LINKS - internal -->

[aks-tutorial-deploy-app]: ./tutorial-kubernetes-deploy-application.md
[aks-tutorial-prepare-acr]: ./tutorial-kubernetes-prepare-acr.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[az ad sp create-for-rbac]: https://docs.azure.cn/cli/ad/sp#az_ad_sp_create_for_rbac
[az acr show]: https://docs.azure.cn/cli/acr#az_acr_show
[az role assignment create]: https://docs.azure.cn/cli/role/assignment#az_role_assignment_create
[az aks create]: https://docs.azure.cn/cli/aks#az_aks_create
[az aks install-cli]: https://docs.azure.cn/cli/aks#az_aks_install_cli
[az aks get-credentials]: https://docs.azure.cn/cli/aks#az_aks_get_credentials
[azure-cli-install]: https://docs.azure.cn/cli/install-azure-cli
[container-registry-integration]: ./cluster-container-registry-integration.md
[quotas-skus-regions]: quotas-skus-regions.md

<!-- Update_Description: update meta properties, wording update, update link -->