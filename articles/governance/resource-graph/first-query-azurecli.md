---
title: 快速入门：第一个 Azure CLI 查询
description: 本快速入门介绍为 Azure CLI 启用 Resource Graph 扩展并运行第一个查询的步骤。
ms.author: v-tawe
origin.date: 08/10/2020
ms.date: 09/15/2020
ms.topic: quickstart
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6653ad2f3246c65fe4f7b9c615f1ba4540668ce8
ms.sourcegitcommit: 75299b1cb5540a11149f320edaae82ae8c03c16b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90523179"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-azure-cli"></a>快速入门：使用 Azure CLI 运行你的第一个 Resource Graph 查询

使用 Azure Resource Graph 的第一步是确保为 [Azure CLI](/cli/) 安装了该扩展。 本快速入门将指导你完成将该扩展添加到 Azure CLI 安装的过程。 可以通过安装在本地的 Azure CLI 使用该扩展。

在此过程结束时，你已将该扩展添加到所选的 Azure CLI 安装中，并将运行你的第一个 Resource Graph 查询。

## <a name="prerequisites"></a>先决条件

如果没有 Azure 订阅，请在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="add-the-resource-graph-extension"></a>添加 Resource Graph 扩展

若要使 Azure CLI 能够查询 Azure Resource Graph，则必须添加该扩展。 此扩展适用于可以使用 Azure CLI 的任何位置，包括 [Windows 10 上的 bash](/windows/wsl/install-win10)、[Azure CLI Docker 映像](https://hub.docker.com/r/microsoft/azure-cli/)或在本地安装。

1. 请确保安装最新的 Azure CLI（至少为 2.0.76）。 若尚未安装，请遵循[这些说明](/cli/install-azure-cli-windows?view=azure-cli-latest)。

1. 在所选的 Azure CLI 环境中，使用以下命令导入该扩展：

   ```azurecli
   # Add the Resource Graph extension to the Azure CLI environment
   az extension add --name resource-graph
   ```

1. 验证该扩展是否已安装以及是否为预期的版本（至少为 **1.0.0**）：

   ```azurecli
   # Check the extension list (note that you may have other extensions installed)
   az extension list

   # Run help for graph query options
   az graph query -h
   ```

## <a name="run-your-first-resource-graph-query"></a>运行首个 Resource Graph 查询

将 Azure CLI 扩展添加到所选环境中后，即可尝试一个简单的 Resource Graph 查询。 该查询将返回前五个 Azure 资源，以及每个资源的名称和资源类型 。

1. 使用 `graph` 扩展和 `query` 命令运行你的第一个 Azure Resource Graph 查询：

   ```azurecli
   # Login first with az login

   # Run Azure Resource Graph query
   az graph query -q 'Resources | project name, type | limit 5'
   ```

   > [!NOTE]
   > 由于此查询示例未提供排序修饰符（例如 `order by`），因此多次运行此查询可能会为每个请求生成一组不同的资源。

1. 将查询更新为 `order by` Name 属性：

   ```azurecli
   # Run Azure Resource Graph query with 'order by'
   az graph query -q 'Resources | project name, type | limit 5 | order by name asc'
   ```

   > [!NOTE]
   > 与第一个查询一样，多次运行此查询可能会为每个请求生成一组不同的资源。 查询命令的顺序非常重要。 在本例中，`order by` 位于 `limit` 之后。 命令按此顺序执行，首先会限制查询结果，然后对它们进行排序。

1. 将查询更新为先 `order by` Name 属性，然后再 `limit` 为前五个结果：

   ```azurecli
   # Run Azure Resource Graph query with `order by` first, then with `limit`
   az graph query -q 'Resources | project name, type | order by name asc | limit 5'
   ```

假设环境中没有任何变化，则多次运行最后一个查询时，返回的结果将是一致的且按 Name 属性排序，但仍限制为前五个结果。

## <a name="clean-up-resources"></a>清理资源

如果希望从 Azure CLI 环境中删除 Resource Graph 扩展，可使用以下命令：

```azurecli
# Remove the Resource Graph extension from the Azure CLI environment
az extension remove -n resource-graph
```

## <a name="next-steps"></a>后续步骤

本快速入门介绍了如何将 Resource Graph 扩展添加到 Azure CLI 环境并运行第一个查询。 若要详细了解 Resource Graph 语言，请继续阅读查询语言详细信息页。

> [!div class="nextstepaction"]
> [获取有关查询语言的详细信息](./concepts/query-language.md)
