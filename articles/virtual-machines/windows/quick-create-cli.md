---
title: 快速入门 - 使用 Azure CLI 创建 Windows VM
description: 本快速入门介绍如何使用 Azure CLI 创建 Windows 虚拟机
ms.service: virtual-machines-windows
ms.topic: quickstart
ms.workload: infrastructure
origin.date: 07/02/2019
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: no
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 9f3c2a8c794e738a3e5c22df618bef8ce8f3914c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106287"
---
# <a name="quickstart-create-a-windows-virtual-machine-with-the-azure-cli"></a>快速入门：使用 Azure CLI 创建 Windows 虚拟机

Azure CLI 用于从命令行或脚本创建和管理 Azure 资源。 本快速入门展示了如何使用 Azure CLI 在 Azure 中部署运行 Windows Server 2016 的虚拟机 (VM)。 若要查看运行中的 VM，可以通过 RDP 登录到该 VM 并安装 IIS Web 服务器。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="launch-azure-local-shell"></a>启动 Azure 本地 Shell

<!--MOONCAKE: az cli 2.0 is put next paragrapgh-->

可以启动 Azure 本地 CLI 控制台以运行以下脚本。

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="create-a-resource-group"></a>创建资源组

使用“[az group create](https://docs.azure.cn/cli/group?view=azure-cli-latest#az-group-create)”命令创建资源组。 Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 以下示例在“chinaeast”  位置创建名为“myResourceGroup”  的资源组：

```azurecli
az group create --name myResourceGroup --location chinaeast
```

## <a name="create-virtual-machine"></a>创建虚拟机

使用 [az vm create](https://docs.azure.cn/cli/vm?view=azure-cli-latest#az-vm-create) 创建 VM。 以下示例创建一个名为 myVM 的 VM。 此示例使用 azureuser 作为管理用户名。 

你将需要提供符合 [Azure VM 密码要求](./faq.md#what-are-the-password-requirements-when-creating-a-vm
)的密码。 使用以下示例时，系统将提示你在命令行中输入密码。 你还可以在 `--admin-password` 参数中添加密码值。 用户名和密码将在以后连接到 VM 时使用。

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2016datacenter \
    --admin-username azureuser 
```

创建 VM 和支持资源需要几分钟时间。 以下示例输出表明 VM 创建操作已成功。

```output
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "chinaeast",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroup"
}
```

记下 VM 输出中自己的 `publicIpAddress`。 在后续步骤中，将使用此地址访问 VM。

## <a name="open-port-80-for-web-traffic"></a>为 Web 流量打开端口 80

默认情况下，在 Azure 中创建 Windows VM 时仅会打开 RDP 连接。 请使用 [az vm open-port](https://docs.azure.cn/cli/vm?view=azure-cli-latest#az-vm-open-port) 打开 TCP 端口 80 以供 IIS Web 服务器使用：

```azurecli
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## <a name="connect-to-virtual-machine"></a>连接到虚拟机

使用以下命令从本地计算机创建远程桌面会话。 将 IP 地址替换为你的 VM 的公共 IP 地址。 出现提示时，输入创建 VM 时使用的凭据：

```powershell
mstsc /v:publicIpAddress
```

## <a name="install-web-server"></a>安装 Web 服务器

若要查看运行中的 VM，请安装 IIS Web 服务器。 在 VM 中打开 PowerShell 提示符并运行以下命令：

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

完成后，关闭到 VM 的 RDP 连接。

## <a name="view-the-web-server-in-action"></a>查看运行中的 Web 服务器

如果 IIS 已安装，并且 VM 上的端口 80 已对 Internet 开放， 则可以使用所选的 Web 浏览器查看默认的 IIS 欢迎页。 使用上一步中获取的 VM 的公共 IP 地址。 以下示例展示了默认 IIS 网站：

:::image type="content" source="./media/quick-create-powershell/default-iis-website.png" alt-text="IIS 默认站点":::

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组、VM 和所有相关的资源，可以使用 [az group delete](https://docs.azure.cn/cli/group?view=azure-cli-latest#az-group-delete) 命令将其删除：

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你部署了简单的虚拟机，打开了 Web 流量的网络端口，并安装了一个基本 Web 服务器。 若要深入了解 Azure 虚拟机，请继续学习 Windows VM 教程。

> [!div class="nextstepaction"]
> [Azure Windows 虚拟机教程](./tutorial-manage-vm.md)

<!-- Update_Description: update meta properties, wording update, update link -->