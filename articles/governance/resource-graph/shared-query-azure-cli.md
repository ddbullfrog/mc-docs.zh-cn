---
title: 快速入门：使用 Azure CLI 创建共享查询
description: 本快速入门将按照步骤为 Azure CLI 启用 Resource Graph 扩展，并创建共享查询。
origin.date: 10/14/2020
ms.date: 11/06/2020
ms.author: v-tawe
ms.topic: quickstart
ms.openlocfilehash: 5b5c70ba3fde33b1ab8c8169dfcd3a92030314bd
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328499"
---
# <a name="quickstart-create-a-resource-graph-shared-query-using-azure-cli"></a>快速入门：使用 Azure CLI 创建 Resource Graph 共享查询

使用 Azure Resource Graph 的第一步是确保为 [Azure CLI](/cli/) 安装了该扩展。 本快速入门将指导你完成将该扩展添加到 Azure CLI 安装的过程。 可以通过安装在本地的 Azure CLI 或通过 [Azure Cloud Shell](https://shell.azure.com) 使用该扩展。

在此过程结束时，你应已将该扩展添加到所选的 Azure CLI 安装中，并可创建一个 Resource Graph 共享查询。

## <a name="prerequisites"></a>先决条件

如果没有 Azure 订阅，请在开始前创建一个[试用帐户](https://wd.azure.cn/pricing/1rmb-trial/)。

<!-- [!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)] -->

## <a name="add-the-resource-graph-extension"></a>添加 Resource Graph 扩展

若要使 Azure CLI 能够用于 Azure Resource Graph，则必须添加该扩展。 此扩展适用于可以使用 Azure CLI 的任何位置，包括 [Windows 10 上的 bash](https://docs.microsoft.com/windows/wsl/install-win10)、[Cloud Shell](https://shell.azure.com)（独立的或位于门户内的）、[Azure CLI Docker 映像](https://hub.docker.com/_/microsoft-azure-cli)，也适用于在本地安装了 Azure CLI 的任何位置。

1. 请确保安装最新的 Azure CLI（至少为 2.8.0）。 若尚未安装，请遵循[这些说明](/cli/install-azure-cli-windows)。

1. 在所选的 Azure CLI 环境中，使用 [az extension add](https://docs.microsoft.com/cli/azure/extension#az-extension-add)，通过以下命令导入 Resource Graph 扩展：

   ```azurecli
   # Add the Resource Graph extension to the Azure CLI environment
   az extension add --name resource-graph
   ```

1. 使用 [az extension list](https://docs.microsoft.com/cli/azure/extension#az-extension-list) 验证该扩展是否已安装以及是否为预期版本（至少为 1.1.0）：

   ```azurecli
   # Check the extension list (note that you may have other extensions installed)
   az extension list

   # Run help for graph query options
   az graph query -h
   ```

## <a name="create-a-resource-graph-shared-query"></a>创建 Resource Graph 共享查询

将 Azure CLI 扩展添加到所选环境中后，即可使用 Resource Graph 查询。 共享查询是一个 Azure 资源管理器对象，你可授予该对象权限或在 Azure Resource Graph Explorer 中运行该对象。 该查询汇总了按“位置”分组的所有资源。

1. 使用 [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) 创建资源组，以存储 Azure Resource Graph 共享查询。 此资源组名为 `resource-graph-queries`，并位于 `chinaeast2`。

   ```azurecli
   # Login first with az login if not using Cloud Shell

   # Create the resource group
   az group create --name 'resource-graph-queries' --location 'chinaeast2'
   ```

1. 使用 `graph` 扩展和 [az graph shared-query create](https://docs.microsoft.com/cli/azure/ext/resource-graph/graph/shared-query#ext-resource-graph-az-graph-shared-query-create) 命令创建 Azure Resource Graph 共享查询：

   ```azurecli
   # Create the Azure Resource Graph shared query
   az graph shared-query create --name 'Summarize resources by location' \
      --description 'This shared query summarizes resources by location for a pinnable map graphic.' \
      --graph-query 'Resources | summarize count() by location' \
      --resource-group 'resource-graph-queries'
   ```

1. 列出新资源组中的共享查询。 [az graph shared-query list](https://docs.microsoft.com/cli/azure/ext/resource-graph/graph/shared-query#ext-resource-graph-az-graph-shared-query-list) 命令返回值数组。

   ```azurecli
   # List all the Azure Resource Graph shared queries in a resource group
   az graph shared-query list --resource-group 'resource-graph-queries'
   ```

1. 若要仅获取单个共享查询结果，请使用 [az graph shared query show](https://docs.microsoft.com/cli/azure/ext/resource-graph/graph/shared-query#ext-resource-graph-az-graph-shared-query-show) 命令。

   ```azurecli
   # Show a specific Azure Resource Graph shared query
   az graph shared-query show --resource-group 'resource-graph-queries' \
      --name 'Summarize resources by location'
   ```

1. 使用 [az graph query](https://docs.microsoft.com/cli/azure/ext/resource-graph/graph#ext-resource-graph-az-graph-query) 命令中的 `{{shared-query-uri}}` 语法在 Azure CLI 中运行共享查询。
   首先，从前面的 `show` 命令结果中复制 `id` 字段。 将示例中 `shared-query-uri` 文本替换为 `id` 字段中的值，但保留周围的 `{{` 和 `}}` 字符。

   ```azurecli
   # Run a Azure Resource Graph shared query
   az graph query --graph-query "{{shared-query-uri}}"
   ```

   > [!NOTE]
   > `{{shared-query-uri}}` 语法是预览功能。

另一种查找 Resource Graph 共享查询的方法是通过 Azure 门户。 在门户中，使用搜索栏搜索“Resource Graph 查询”。 选择共享查询。 在“概述”页上，“查询”选项卡显示已保存的查询 。 在 [Resource Graph Explorer](./first-query-portal.md) 中单击“编辑”按钮将其打开。

## <a name="clean-up-resources"></a>清理资源

如果要从 Azure CLI 环境中删除 Resource Graph 共享查询、资源组和扩展，可以使用以下命令执行此操作：

- [az graph shared-query delete](https://docs.microsoft.com/cli/azure/ext/resource-graph/graph/shared-query#ext-resource-graph-az-graph-shared-query-delete)
- [az group delete](https://docs.microsoft.com/cli/azure/group#az-group-delete)
- [az extension remove](https://docs.microsoft.com/cli/azure/extension#az-extension-remove)

```azurecli
# Delete the Azure Resource Graph shared query
az graph shared-query delete --resource-group 'resource-graph-queries' \
   --name 'Summarize resources by location'

# Remove the resource group
# WARNING: This command deletes ALL resources you've added to this resource group without prompting for confirmation
az group delete --resource-group 'resource-graph-queries' --yes

# Remove the Azure Resource Graph extension from the Azure CLI environment
az extension remove -n resource-graph
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已将 Resource Graph 扩展添加到 Azure CLI 环境并创建了一个共享查询。 若要详细了解 Resource Graph 语言，请继续阅读查询语言详细信息页。

> [!div class="nextstepaction"]
> [获取有关查询语言的详细信息](./concepts/query-language.md)