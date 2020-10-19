---
title: 发现资源属性
description: 描述如何搜索资源属性。
ms.topic: conceptual
origin.date: 06/10/2020
author: rockboyfor
ms.date: 10/12/2020
ms.testscope: yes
ms.testdate: 08/24/2020
ms.author: v-yeche
ms.openlocfilehash: 42d213495cdeb65f4372d444b23c61c21da011e3
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937539"
---
# <a name="discover-resource-properties"></a>发现资源属性

创建资源管理器模板之前，需要了解有哪些资源类型可用以及要在模板中使用哪些值。 本文介绍了一些可用于查找要在模板中包含的属性的方法。

## <a name="find-resource-provider-namespaces"></a>查找资源提供程序命名空间

ARM 模板中的资源是使用资源提供程序命名空间和资源类型来定义的。 例如，Microsoft.Storage/storageAccounts 是存储帐户资源类型的完整名称。 Microsoft.Storage 是命名空间。 如果你还不知道要使用的资源类型的命名空间，请参阅 [Azure 服务的资源提供程序](../management/azure-services-resource-providers.md)。

:::image type="content" source="./media/view-resources/resource-provider-namespace-and-azure-service-mapping.png" alt-text="资源管理器资源提供程序命名空间映射":::

## <a name="export-templates"></a>导出模板

为现有资源获取模板属性的最简单方法就是导出模板。 有关详细信息，请参阅[在 Azure 门户中将单个资源和多个资源导出到模板](./export-template-portal.md)。

## <a name="use-resource-manager-tools-extension"></a>使用资源管理器工具扩展

Visual Studio Code 和 Azure 资源管理器工具扩展有助于确切了解每种资源类型需要哪些属性。 它们提供 Intellisense 和代码片段，可简化在模板中定义资源的过程。 有关详细信息，请参阅[快速入门：使用 Visual Studio Code 创建 Azure 资源管理器模板](./quickstart-create-templates-use-visual-studio-code.md#add-an-azure-resource)。

以下屏幕截图显示了如何将存储帐户资源添加到模板：

:::image type="content" source="./media/view-resources/resource-manager-tools-extension-snippets.png" alt-text="资源管理器资源提供程序命名空间映射":::

该扩展还提供了用于配置属性的选项列表。

:::image type="content" source="./media/view-resources/resource-manager-tools-extension-configurable-properties.png" alt-text="资源管理器资源提供程序命名空间映射":::

<!--Not Available on ## Use template reference-->

## <a name="use-resource-explorer"></a>使用资源浏览器

资源浏览器是嵌入在 Azure 门户中的。 在使用此方法之前，需要先有一个存储帐户。 如果还没有，请选择以下按钮来创建一个：

[![“部署到 Azure”](https://aka.ms/deploytoazurebutton)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-storage-account-create%2fazuredeploy.json)

1. 登录到 [Azure 门户](https://portal.azure.cn)。
1. 在搜索框中，输入“资源浏览器”，然后选择“资源浏览器”。

    :::image type="content" source="./media/view-resources/azure-portal-resource-explorer.png" alt-text="资源管理器资源提供程序命名空间映射":::

1. 从左侧展开“订阅”，然后展开你的 Azure 订阅。 在“提供程序”或“ResourceGroups”下可找到存储帐户。

    :::image type="content" source="./media/view-resources/azure-portal-resource-explorer-home.png" alt-text="资源管理器资源提供程序命名空间映射":::

    - 提供程序：展开“提供程序” -> “Microsoft.Storage” -> “storageAccounts”，然后选择你的存储帐户。
    - ResourceGroups：选择包含该存储帐户的资源组，选择“资源”，然后选择该存储帐户。

    在右侧，可以看到现有存储帐户的 SKU 配置，如下所示：

    :::image type="content" source="./media/view-resources/azure-portal-resource-explorer-sku.png" alt-text="资源管理器资源提供程序命名空间映射":::

<!--Not Available on ## Use Resources.azure.com-->

## <a name="next-steps"></a>后续步骤

本文介绍了如何查找模板架构信息。 若要详细了解如何创建资源管理器模板，请参阅[了解 ARM 模板的结构和语法](./template-syntax.md)。

<!-- Update_Description: update meta properties, wording update, update link -->