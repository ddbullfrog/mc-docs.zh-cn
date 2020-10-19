---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/22/2020
title: 工作区库 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用和管理工作区库。
ms.openlocfilehash: 123b28ba6f788d0c9d638b331306351a5f0bfb30
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937762"
---
# <a name="workspace-libraries"></a>工作区库

工作区库充当本地存储库，你可以从中创建群集安装库。 工作区库可能是你的组织创建的自定义代码，也可能是你的组织已经标准化的开源库的特定版本。

必须先在群集上安装工作区库，然后才能将其用于笔记本或作业。

工作区中的所有用户均可使用共享文件夹中的工作区库，而某个用户文件夹中的工作区库仅该用户可用。

## <a name="create-a-workspace-library"></a>创建工作区库

1. 右键单击用于存储该库的工作区文件夹。
2. 选择“创建”>“库”。

   > [!div class="mx-imgBorder"]
   > ![创建库](../_static/images/libraries/create-library.png)

   将显示“创建库”对话框。

   > [!div class="mx-imgBorder"]
   > ![库选项](../_static/images/libraries/select-library-azure.png)

3. 选择“库源”并按照相应的过程操作：
   * [上传库](#uploading-libraries)
   * [引用已上传的库](#referencing-libraries)
   * [PyPI 包](#pypi-libraries)
   * [Maven 包](#maven-libraries)
   * [CRAN 包](#cran-libraries)

### <a name="upload-a-jar-python-egg-or-python-wheel"></a><a id="upload-a-jar-python-egg-or-python-wheel"> </a><a id="uploading-libraries"> </a>上传 Jar、Python Egg 或 Python Wheel

1. 在“库源”按钮列表中，选择“上传”。
2. 选择“Jar”、“Python Egg”或“Python Whl”  。
3. 选择性地输入库名称。
4. 将 Jar、Egg 或 Whl 拖到下拉框中，或单击下拉框，然后导航到文件。 该文件将上传到 `dbfs:/FileStore/jars`。
5. 单击“创建”。 将显示“库状态”屏幕。
6. 选择性地[将库安装到群集上](cluster-libraries.md#install-workspace-libraries)。

### <a name="reference-an-uploaded-jar-python-egg-or-python-wheel"></a><a id="reference-an-uploaded-jar-python-egg-or-python-wheel"> </a><a id="referencing-libraries"> </a>引用已上传的 Jar、Python Egg 或 Python Wheel

如果已将 Jar、Egg 或 Wheel 上传到对象存储，则可以在工作区库中引用。

可以在 DBFS 中选择一个库。

1. 在“库源”按钮列表中，选择“DBFS”。
2. 选择“Jar”、“Python Egg”或“Python Whl”  。
3. 选择性地输入库名称。
4. 指定库的 DBFS 路径。
5. 单击“创建”。 将显示“库状态”屏幕。
6. 选择性地[将库安装到群集上](cluster-libraries.md#install-workspace-libraries)。

### <a name="pypi-package"></a><a id="pypi-libraries"> </a><a id="pypi-package"> </a>PyPI 包

1. 在“库源”按钮列表中，选择“PyPI”。
2. 输入 PyPI 包名称。 若要安装特定版本的库，请对该库使用此格式：`<library>==<version>`。 例如，`scikit-learn==0.19.1`。
3. 在“存储库”字段中，选择性地输入 PyPI 存储库 URL。
4. 单击“创建”。 将显示“库状态”屏幕。
5. 选择性地[将库安装到群集上](cluster-libraries.md#install-workspace-libraries)。

### <a name="maven-or-spark-package"></a><a id="maven-libraries"> </a><a id="maven-or-spark-package"> </a>Maven 或 Spark 包

1. 在“库源”按钮列表中，选择“Maven”。
2. 指定 Maven 坐标。 执行下列操作之一：
   * 在“坐标”字段中，输入要安装的库的 Maven 坐标。 Maven 坐标的格式为 `groupId:artifactId:version`；例如 `com.databricks:spark-avro_2.10:1.0.0`。
   * 如果不知道确切的坐标，请输入库名称，然后单击“搜索包”。 将显示匹配的包的列表。 若要显示有关包的详细信息，请单击其名称。 可以按名称、组织和评级对包进行排序。 还可以通过在搜索栏中编写查询来筛选结果。 结果将自动刷新。
     1. 在左上角的下拉列表中选择“Maven Central”或“Spark 包 ” 。
     1. 可选择在“发布”列中选择包版本。
     1. 单击包旁边的“+ 选择”。 将用所选包和版本填充该“坐标”字段。
3. 在“存储库”字段中，选择性地输入 Maven 存储库 URL。

   > [!NOTE]
   >
   > 不支持内部 Maven 存储库。

4. 在“排除项”字段中，选择性地提供要排除的依赖项的 `groupId` 和 `artifactId`；例如 `log4j:log4j`。
5. 单击“创建”。 将显示“库状态”屏幕。
6. 选择性地[将库安装到群集上](cluster-libraries.md#install-libraries)。

### <a name="cran-package"></a><a id="cran-libraries"> </a><a id="cran-package"> </a>CRAN 包

1. 在“库源”按钮列表中，选择“CRAN”。
2. 在“包”字段中，输入包的名称。
3. 在“存储库”字段中，选择性地输入 CRAN 存储库 URL。
4. 单击“创建”。 将显示“库详细信息”屏幕。
5. 选择性地[将库安装到群集上](cluster-libraries.md#install-libraries)。

> [!NOTE]
>
> CRAN 镜像提供库的最新版本。 因此，如果在不同的时间将库附加到不同的群集，则最终可能会得到不同版本的 R 包。 若要了解如何在 Databricks 上管理和修复 R 包版本，请参阅[知识库](/azure/databricks/kb/r/pin-r-packages)。

## <a name="view-workspace-library-details"></a><a id="view-library"> </a><a id="view-workspace-library-details"> </a>查看工作区库详细信息

1. 转到包含该库的工作区文件夹。
2. 单击库名称。

“库详细信息”页面显示该库运行中的群集及其[安装状态](../dev-tools/api/latest/libraries.md#managedlibrarieslibraryinstallstatus)。 如果已安装库，则页面包含指向包主机的链接。 如果已上传库，则页面将显示指向已上传的包文件的链接。

## <a name="move-a-workspace-library"></a>移动工作区库

1. 转到包含该库的工作区文件夹。
2. 单击库名称右边的下拉箭头![菜单下拉箭头](../_static/images/menu-dropdown.png)，然后选择“移动”。 将显示文件夹浏览器。
3. 单击目标文件夹。
4. 单击“选择”。
5. 单击“确认并移动”。

## <a name="delete-a-workspace-library"></a>删除工作区库

> [!IMPORTANT]
>
> 删除工作区库之前，应将其从所有群集中[卸载](cluster-libraries.md#uninstall-libraries)。

若要删除工作区库，请执行以下操作：

1. 将库移动到“回收站”文件夹。
2. 永久删除“回收站”文件夹中的库，或清空“回收站”文件夹。