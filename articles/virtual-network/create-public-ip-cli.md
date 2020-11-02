---
title: 创建公共 IP - Azure CLI
description: 了解如何使用 Azure CLI 创建公共 IP
services: virtual-network
documentationcenter: na
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 08/28/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 10/05/2020
ms.author: v-yeche
ms.openlocfilehash: 2f0b95bc605b7c35874d02edba745a015a6b6e26
ms.sourcegitcommit: 1f933e4790b799ceedc685a0cea80b1f1c595f3d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92628182"
---
<!--Verified Successfully-->
<!--Remove the part of Availability Zones-->
# <a name="quickstart-create-a-public-ip-address-using-azure-cli"></a>快速入门：使用 Azure CLI 创建公共 IP 地址

本文介绍了如何使用 Azure CLI 来创建公共 IP 地址。 若要详细了解这可能关联到哪些资源，以及基本 SKU 和标准 SKU 之间的差异和其他相关信息，请参阅[公共 IP 地址](/virtual-network/public-ip-addresses)。  对于此示例，我们只重点介绍 IPv4 地址；有关 IPv6 地址的详细信息，请参阅[适用于 Azure VNet 的 IPv6](/virtual-network/ipv6-overview)。

## <a name="prerequisites"></a>先决条件

- 已安装在本地的 Azure CLI 或 Azure 本地的 Shell

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)] 

如果选择在本地安装并使用 CLI，本快速入门要求 Azure CLI 2.0.28 或更高版本。 若要查找版本，请运行 `az --version`。 如需进行安装或升级，请参阅[安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli)。

## <a name="create-a-resource-group"></a>创建资源组

Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。

使用 [az group create](https://docs.azure.cn/cli/group#az_group_create) 在 chinaeast2 位置创建名为“myResourceGroup”的资源组 。

```azurecli
  az group create \
    --name myResourceGroup \
    --location chinaeast2
```
---

<!--Not Available on Availability Zones-->
<!--Not Available on Availability Zones-->

<a name="option-create-public-ip-basic"></a>
## <a name="basic-sku"></a>基本 SKU

使用 [az network public-ip create](https://docs.azure.cn/cli/network/public-ip#az-network-public-ip-create) 在 myResourceGroup 中创建名为“myBasicPublicIP”的基本静态公共 IP 地址 。  

<!--Not Available on Availability Zones-->

```azurecli
  az network public-ip create \
    --resource-group myResourceGroup \
    --name myBasicPublicIP \
    --sku Standard
    --allocation-method Static
```
如果可以接受 IP 地址随时间的推移发生更改，可以通过将 allocation-method 更改为“Dynamic”来选择动态 IP 分配。

---

## <a name="additional-information"></a>其他信息 

若要详细了解以上所列的各个变量，请参阅[管理公共 IP 地址](/virtual-network/virtual-network-public-ip-address#create-a-public-ip-address)。

## <a name="next-steps"></a>后续步骤
- [将公共 IP 地址关联到虚拟机](/virtual-network/associate-public-ip-address-vm#azure-portal)。
- 详细了解 Azure 中的[公共 IP 地址](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses)。
- 详细了解所有[公共 IP 地址设置](virtual-network-public-ip-address.md#create-a-public-ip-address)。

<!-- Update_Description: update meta properties, wording update, update link -->