---
title: 使用 Azure CLI 创建 IoT 中心 | Microsoft Docs
description: 如何使用 Azure CLI 创建 Azure IoT 中心。
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
origin.date: 08/23/2018
ms.author: v-yiso
ms.date: 10/08/2018
ms.openlocfilehash: c26e5379da2a6db7b4002e4ea4577f09a4a30970
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937321"
---
# <a name="create-an-iot-hub-using-the-azure-cli"></a>使用 Azure CLI 创建 IoT 中心

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

本文介绍如何使用 Azure CLI 创建 IoT 中心。

## <a name="prerequisites"></a>必备条件

若要完成本操作说明，需要 Azure 订阅。 如果没有 Azure 订阅，可在开始前创建一个 [试用帐户](https://www.azure.cn/pricing/1rmb-trial) 。


## <a name="sign-in-and-set-your-azure-account"></a>登录并设置 Azure 帐户

登录到 Azure 帐户，并选择订阅。

1. 在命令提示符中，运行 [login 命令][lnk-login-command]：
    
    ```azurecli
    az login
    ```

    按照说明使用代码进行身份验证，并通过 Web 浏览器登录 Azure 帐户。

## <a name="create-an-iot-hub"></a>创建 IoT 中心

使用 Azure CLI 创建资源组，并添加 IoT 中心。

1. 创建 IoT 中心时，必须在资源组中创建它。 使用现有资源组，或运行以下[命令创建资源组][lnk-az-resource-command]：

    ```azurecli
     az group create --name {your resource group name} --location westus
    ```

    > [!TIP]
    > 上一示例在美国西部位置创建资源组。 可以通过运行以下命令来查看可用位置列表： 
    >
    > ```azurecli
    >az account list-locations -o table
    >```
    >

2. 运行以下[命令，使用 IoT 中心的全局唯一名称在资源组中创建 IoT 中心][lnk-az-iot-command]：
    
   ```azurecli
   az iot hub create --name {your iot hub name} \
      --resource-group {your resource group name} --sku S1
   ```

   [!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]


> [!NOTE]
> 上一命令在计费的 S1 定价层中创建 IoT 中心。 有关详细信息，请参阅 [Azure IoT 中心定价][lnk-iot-pricing]。
>
>

## <a name="remove-an-iot-hub"></a>删除 IoT 中心

可使用 Azure CLI [删除单个资源][lnk-az-resource-command]（例如 IoT 中心），或删除资源组及其所有资源（包括任何 IoT 中心）。

若要[删除 IoT 中心](/cli/iot/hub#az-iot-hub-delete)，请运行以下命令：

```azurecli
az iot hub delete --name {your iot hub name} -\
  -resource-group {your resource group name}
```

若要[删除资源组](/cli/group#az-group-delete)及其所有资源，请运行以下命令：

```azurecli
az group delete --name {your resource group name}
```

## <a name="next-steps"></a>后续步骤

若要详细了解如何使用 IoT 中心，请参阅以下文章：

* [IoT 中心开发人员指南](iot-hub-devguide.md)
* [使用 Azure 门户管理 IoT 中心](iot-hub-create-through-portal.md)

<!-- Links -->
[lnk-free-trial]: https://www.azure.cn/pricing/1rmb-trial/
[lnk-CLI-install]: https://docs.azure.cn/zh-cn/cli/install-az-cli2?view=azure-cli-lastest
[lnk-login-command]: https://docs.azure.cn/zh-cn/cli/get-started-with-azure-cli?view=azure-cli-lastest
[lnk-az-account-command]: https://docs.azure.cn/zh-cn/cli/account?view=azure-cli-latest
[lnk-az-register-command]: https://docs.azure.cn/zh-cn/cli/provider?view=azure-cli-latest
[lnk-az-addcomponent-command]: https://docs.azure.cn/zh-cn/cli/component?view=azure-cli-latest
[lnk-az-resource-command]: https://docs.azure.cn/zh-cn/cli/resource?view=azure-cli-latest
[lnk-az-iot-command]: https://docs.azure.cn/zh-cn/cli/iot?view=azure-cli-latest
[lnk-iot-pricing]: https://www.azure.cn/pricing/details/iot-hub/
[lnk-devguide]: ./iot-hub-devguide.md
[lnk-portal]: ./iot-hub-create-through-portal.md

<!--Update_Description: update meta data only-->