---
title: 创建多个资源实例
description: 了解如何创建 Azure 资源管理器模板，以用于创建多个 Azure 资源实例。
origin.date: 04/23/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: no
ms.testdate: ''
ms.topic: tutorial
ms.author: v-yeche
ms.custom: devx-track-azurecli
ms.openlocfilehash: 8b6b8546928d1d6e97050abb480b2d03ca3ff31c
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470072"
---
# <a name="tutorial-create-multiple-resource-instances-with-arm-templates"></a>教程：使用 ARM 模板创建多个资源实例

了解如何在 Azure 资源管理器 (ARM) 模板中进行迭代操作，以创建 Azure 资源的多个实例。 在本教程中，你将修改一个模板，以便创建三个存储帐户实例。

:::image type="content" source="./media/template-tutorial-create-multiple-instances/resource-manager-template-create-multiple-instances-diagram.png" alt-text="“Azure 资源管理器创建多个实例”示意图":::

本教程涵盖以下任务：

> [!div class="checklist"]
> * 打开快速入门模板
> * 编辑模板
> * 部署模板

如果没有 Azure 订阅，请在开始前[创建一个试用帐户](https://www.azure.cn/pricing/1rmb-trial/)。

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：

* 包含资源管理器工具扩展的 Visual Studio Code。 请参阅[快速入门：使用 Visual Studio Code 创建 Azure 资源管理器模板](quickstart-create-templates-use-visual-studio-code.md)。

## <a name="open-a-quickstart-template"></a>打开快速入门模板

[Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates/)是 ARM 模板的存储库。 无需从头开始创建模板，只需找到一个示例模板并对其自定义即可。 本快速入门中使用的模板称为[创建标准存储帐户](https://github.com/Azure/azure-quickstart-templates/tree/master/101-storage-account-create/)。 该模板定义 Azure 存储帐户资源。

1. 在 Visual Studio Code 中，选择“文件”>“打开文件”。 
2. 在“文件名”中粘贴以下 URL：

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
    ```
3. 选择“打开”以打开该文件。
4. 在模板中定义了一个“Microsoft.Storage/storageAccounts”资源。
    
   <!-- Not Available on [template reference](https://docs.microsoft.com/azure/templates/Microsoft.Storage/storageAccounts)-->

5. 选择“文件”>“另存为”，将该文件作为 **azuredeploy.json** 保存到本地计算机。 

## <a name="edit-the-template"></a>编辑模板

现有模板创建一个存储帐户。 请通过自定义模板来创建三个存储帐户。

从 Visual Studio Code 中，进行以下四个更改：

:::image type="content" source="./media/template-tutorial-create-multiple-instances/resource-manager-template-create-multiple-instances.png" alt-text="“Azure 资源管理器创建多个实例”示意图"
      },
      "kind": "StorageV2",
      "copy": {
        "name": "storagecopy",
        "count": 3
      },
      "properties": {}
    }
  ]
}
```

有关创建多个实例的详细信息，请参阅[在 ARM 模板中部署资源或属性的多个实例](./copy-resources.md)

## <a name="deploy-the-template"></a>部署模板

<!--Not Available on [Azure local Shell](https://shell.azure.com)-->
<!--Not Available on Azure China Cloud-->

1. 在本地 Shell 中运行以下命令。 选择用于显示 PowerShell 代码或 CLI 代码的选项卡。

    # <a name="cli"></a>[CLI](#tab/CLI)

    ```azurecli
    echo "Enter a project name that is used to generate resource group name:" &&
    read projectName &&
    echo "Enter the location (i.e. chinaeast):" &&
    read location &&
    resourceGroupName="${projectName}rg" &&
    az group create --name $resourceGroupName --location "$location" &&
    az deployment group create --resource-group $resourceGroupName --template-file "$HOME/azuredeploy.json"
    ```

    # <a name="powershell"></a>[PowerShell](#tab/PowerShell)

    ```azurepowershell
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name"
    $location = Read-Host -Prompt "Enter the location (i.e. chinaeast)"
    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateFile "$HOME/azuredeploy.json"
    ```

    ---

若要列出全部三个存储帐户，请省略 --name 参数：

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
echo "Enter a project name that is used to generate resource group name:" &&
read projectName &&
resourceGroupName="${projectName}rg" &&
az storage account list --resource-group $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name"
$resourceGroupName = "${projectName}rg"

Get-AzStorageAccount -ResourceGroupName $resourceGroupName
Write-Host "Press [ENTER] to continue ..."
```

---

将存储帐户名称与模板中的名称定义进行比较。

## <a name="clean-up-resources"></a>清理资源

不再需要 Azure 资源时，请通过删除资源组来清理部署的资源。

1. 在 Azure 门户上的左侧菜单中选择“资源组”。
2. 在“按名称筛选”字段中输入资源组名称。
3. 选择资源组名称。  应会看到，该资源组中总共有三个资源。
4. 在顶部菜单中选择“删除资源组”。

## <a name="next-steps"></a>后续步骤

在本教程中，你学习了如何创建多个存储帐户实例。  在下一教程中，我们将开发包含多个资源和多个资源类型的模板。 某些资源具有依赖的资源。

> [!div class="nextstepaction"]
> [创建所依赖的资源](./template-tutorial-create-templates-with-dependent-resources.md)

<!-- Update_Description: update meta properties, wording update, update link -->
