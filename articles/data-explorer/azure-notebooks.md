---
title: 使用 Azure Notebooks 分析 Azure 数据资源管理器中的数据
description: 本主题说明如何使用 Azure Notebook 创建查询
author: orspod
ms.author: v-tawe
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: how-to
origin.date: 04/01/2020
ms.date: 09/24/2020
ms.openlocfilehash: 9399aac87eb630d07a78ad377a3e659e1a33e3a4
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146272"
---
# <a name="use-azure-notebooks-to-analyze-data-in-azure-data-explorer"></a>使用 Azure Notebooks 分析 Azure 数据资源管理器中的数据

[Azure Notebooks](https://notebooks.azure.com/) 是一种 Azure 云服务，可简化 [Jupyter Notebook](https://jupyter.org/) 的创建和共享。 使用 Azure Notebooks 可轻松合并文档、代码和运行代码的结果。

> [!Note]
> * 以下过程在 Azure Notebooks 环境中使用 Python 客户端查询 Azure 数据资源管理器。 不过，也可以使用 [KQLmagic](kqlmagic.md) 来查询 Azure 数据资源管理器。
> * 某些用户报告使用 Microsoft Edge 进行身份验证时出现问题；在这些问题得到解决之前，请使用其他浏览器。

## <a name="create-a-project"></a>创建一个项目

1. 在浏览器中打开 [Azure Notebooks](https://notebooks.azure.com/) 并登录。

1. 在标题中选择“我的项目”标签页。 

    :::image type="content" source="media/azurenotebooks/an-myprojects.png" alt-text="“项目”页、“我的项目”选项卡、Microsoft Azure Notebooks、Azure 数据资源管理器" lightbox="media/azurenotebooks/an-myprojects.png#lightbox":::

1. 选择“+ 新建项目”。
    
1. 在“创建新项目”对话框中执行以下操作：
    1. 输入项目名称。
    1. 清除“公共”复选框。
        >[!Important]
        > 如果未清除“公共”复选框，则你的项目会在开放 Internet 上公开。
    1. 选择“创建” 。
    
    ![创建新项目](media/azurenotebooks/an-create-new-project-blank.png)

    此时会自动创建项目 ID。

## <a name="create-a-notebook"></a>创建笔记本

1. 在新项目中，创建笔记本。 笔记本应使用[支持的语言](https://github.com/Azure/azure-kusto-python#minimum-requirements)。
若要创建笔记本，请选择“+ 新建”并选择“笔记本”。

    ![新建笔记本](media/azurenotebooks/an-create-new-notebook-menu.png) 

1. 在“新建笔记本”对话框中，输入笔记本名称。

1. 选择“Python 3.6”，然后选择“新建”。
    
    ![新建笔记本](media/azurenotebooks/an-create-new-notebook.png) 
    
1. 在项目中，选择新笔记本。

    ![选择新笔记本](media/azurenotebooks/an-select-notebook.png)

    等待至 Python 内核初始化。 当实心圆图标变为空心圆图标时，可以继续操作。

    ![内核初始化](media/azurenotebooks/an-python-init-icon.png)

1. 若要导入系统模块，请输入以下命令，然后选择“运行”：
    ```python
    import sys
    sys.version
    ```

    > [!Note]
    > 若要运行某个单元，也可以按 Shift+Enter。

1.  若要导入 SDK 类，请输入以下命令，然后选择“运行”：
    ```python
    from azure.kusto.data.request import KustoClient, KustoConnectionStringBuilder
    ```

1.  若要生成连接字符串并启动 Kusto 客户端，请输入以下命令，然后选择“运行”：  
    ```python
    kcsb = KustoConnectionStringBuilder.with_aad_device_authentication("https://help.kusto.chinacloudapi.cn")
    kc = KustoClient(kcsb)
    kc.execute("Samples", ".show version")
    ```
1. 在提示符下，打开一个新的浏览器标签页并访问 [https://login.partner.microsoftonline.cn/common/oauth2/deviceauth](https://login.partner.microsoftonline.cn/common/oauth2/deviceauth)。
   
1. 输入授权代码，并登录 AAD (@microsoft.com) 帐户。 Kusto 客户端 `kc` 现在可以使用你的身份向 Azure 数据资源管理器进行身份验证。

1. 返回到笔记本以查看身份验证的结果。 

:::image type="content" source="media/azurenotebooks/an-python-commands.png" alt-text="“项目”页、“我的项目”选项卡、Microsoft Azure Notebooks、Azure 数据资源管理器" lightbox="media/azurenotebooks/an-python-commands.png#lightbox":::

## <a name="execute-a-kusto-query"></a>执行 Kusto 查询

1. 输入 Kusto 查询并选择“运行”。 例如：

    ```python
    query= "StormEvents | project State, EventType | take 10"
    response = kc.execute("Samples", query)
    for row in response.primary_results[0]:
        print(", ".join(row))
    ```    

:::image type="content" source="media/azurenotebooks/an-commands.png" alt-text="“项目”页、“我的项目”选项卡、Microsoft Azure Notebooks、Azure 数据资源管理器" lightbox="media/azurenotebooks/an-commands.png#lightbox":::

## <a name="next-steps"></a>后续步骤

* [Kusto-python GitHub 存储库](https://github.com/Azure/azure-kusto-python)
