---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/23/2020
title: 工作区对象访问控制 - Azure Databricks
description: 了解如何控制对 Azure Databricks 工作区对象（例如文件夹、笔记本、MLflow 试验和 MLflow 模型）的访问。
ms.openlocfilehash: b2896a002dcc6ec1cd0d1281ca10134a3378f5e5
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937711"
---
# <a name="workspace-object-access-control"></a>工作区对象访问控制

> [!NOTE]
>
> 访问控制仅在 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)中提供。

默认情况下，除非管理员启用工作区访问控制，否则所有用户均可创建和修改[工作区](../../workspace/index.md)对象（包括文件夹、笔记本、试验和模型）。 在使用工作区对象访问控制的情况下，用户的操作能力取决于单个权限。 本文介绍各个权限以及配置工作区对象访问控制的方式。

Azure Databricks 管理员必须先为工作区启用工作区对象访问控制，然后你才能使用该控制。 请参阅[启用工作区对象访问控制](../../administration-guide/access-control/workspace-acl.md)。

## <a name="folder-permissions"></a><a id="folder-permissions"> </a><a id="permissions"> </a>文件夹权限

可以为[文件夹](../../workspace/workspace-objects.md#folders)分配五个权限级别：“无权限”、“读取”、“运行”、“编辑”和“管理”。 该表列出了每个权限赋予用户的能力。

| 能力                                     | 无权限   | 读取       | 运行     | 编辑    | 管理    |
|---------------------------------------------|------------------|------------|---------|---------|-----------|
| 列出文件夹中的项                        | x                | x          | x       | x       | x         |
| 查看文件夹中的项                        |                  | x          | x       | x       | x         |
| 克隆和导出项                      |                  | x          | x       | x       | x         |
| 创建、导入和删除项            |                  |            |         |         | x         |
| 移动和重命名项                       |                  |            |         |         | x         |
| 更改权限                          |                  |            |         |         | x         |

文件夹中的笔记本和试验继承该文件夹的所有权限设置。 例如，在某个文件夹上拥有“运行”权限的用户对该文件夹中的笔记本拥有“运行”权限。

### <a name="default-folder-permissions"></a>默认的文件夹权限

* 以下权限独立于工作区对象访问控制而存在：
  * 对于 **Workspace** > ![共享图标](../../_static/images/workspace/shared-icon.png) **Shared** 文件夹中的项，所有用户都有“管理”权限。 可以授予针对笔记本和文件夹的“管理”权限，方法是将笔记本和文件夹移到 ![共享图标](../../_static/images/workspace/shared-icon.png) **Shared** 文件夹。
  * 所有用户对自己创建的对象都有“管理”权限。
* 在禁用工作区对象访问控制的情况下，存在以下权限：
  * 对于 **Workspace** 文件夹中的项，所有用户都有“编辑”权限。
* 在[启用工作区对象访问控制](../../administration-guide/access-control/workspace-acl.md#enable-workspace-acl)的情况下，存在以下权限：
  * **Workspace** 文件夹
    * 只有管理员才能在 **Workspace** 文件夹中创建新项目。
    * **Workspace** 文件夹中的现有项 -“管理”。 例如，如果 **Workspace** 文件夹包含 ![Folder](../../_static/images/access-control/folder.png) **Documents** 和 ![Folder](../../_static/images/access-control/folder.png) **Temp** 文件夹，则所有用户都会继续拥有针对这些文件夹的“管理”权限。
    * **Workspace** 文件夹中的新项 -“无权限”。
  * 用户对文件夹中的所有项（包括在设置权限后创建到或移动到文件夹中的项）的权限与用户对该文件夹的权限相同。
  * 用户主目录 - 用户有“管理”权限。 所有其他用户的权限为“无权限”。

## <a name="notebook-permissions"></a>笔记本权限

可以为[笔记本](../../workspace/workspace-assets.md#ws-notebooks)分配五个权限级别：“无权限”、“读取”、“运行”、“编辑”和“管理”。 该表列出了每个权限赋予用户的能力。

| 能力                                     | 无权限   | 读取     | 运行     | 编辑    | 管理   |
|---------------------------------------------|------------------|----------|---------|---------|----------|
| 查看单元                                  |                  | x        | x       | x       | x        |
| 注释                                     |                  | x        | x       | x       | x        |
| 通过 %run 或笔记本工作流来运行          |                  | x        | x       | x       | x        |
| 附加和分离笔记本                 |                  |          | x       | x       | x        |
| 运行命令                                |                  |          | x       | x       | x        |
| 编辑单元                                  |                  |          |         | x       | x        |
| 更改权限                          |                  |          |         |         | x        |

## <a name="configure-notebook-and-folder-permissions"></a>配置笔记本和文件夹权限

> [!NOTE]
>
> 此部分介绍如何使用 UI 来管理权限。 你还可以使用[权限 API](../../_static/api-refs/permissions-azure.yaml)。

1. 打开“权限”对话框：
   * 笔记本 - 在笔记本上下文栏中单击“ ![权限](../../_static/images/access-control/permissions.png) ”。
   * 文件夹 - 在文件夹的下拉菜单中选择“权限”：

   > [!div class="mx-imgBorder"]
   > ![“权限”下拉菜单](../../_static/images/access-control/permission-drop-down.png)

2. 若要向用户或组授予权限，请从“添加用户和组”下拉菜单中选择权限，然后单击“添加”： 

   > [!div class="mx-imgBorder"]
   > ![添加用户](../../_static/images/access-control/add-users.png)

   若要更改用户或组的权限，请从权限下拉菜单中选择新权限：

   > [!div class="mx-imgBorder"]
   > ![更改权限](../../_static/images/access-control/change-permissions.png)

3. 单击“保存更改”保存所做的更改，或单击“取消”放弃所做的更改。

## <a name="mlflow-experiment-permissions"></a>MLflow 试验权限

可以为 [MLflow 试验](../../applications/mlflow/tracking.md#experiments)分配四个权限级别：“无权限”、“读取”、“编辑”和“管理”。  该表列出了每个权限赋予用户的能力。

| 能力                                     | 无权限   | 读取    | 编辑     | 管理   |
|---------------------------------------------|------------------|---------|----------|----------|
| 查看运行信息、搜索、比较运行         |                  | x       | x        | x        |
| 查看、列出和下载运行项目      |                  | x       | x        | x        |
| 创建、删除和还原运行            |                  |         | x        | x        |
| 记录运行参数、指标、标记               |                  |         | x        | x        |
| 记录运行项目                           |                  |         | x        | x        |
| 编辑试验标记                        |                  |         | x        | x        |
| 清除运行和试验                  |                  |         |          | x        |
| 授予权限                           |                  |         |          | x        |

> [!NOTE]
>
> * 只会对存储在由 MLflow 管理的 DBFS 位置中的项目强制实施试验权限。
>   有关详细信息，请参阅 [MLflow 项目权限](#mlflow-artifact-permissions)。
> * 创建、删除和还原试验需要对包含试验的文件夹具有“编辑”或“管理”访问权限。 
> * 可以为试验指定“运行”权限。 它以与“编辑”相同的方式强制实施。

### <a name="configure-mlflow-experiment-permissions"></a>配置 MLflow 试验权限

1. 打开“权限”对话框。 在笔记本上下文栏中单击“ ![权限](../../_static/images/access-control/permissions.png) ”。

   > [!div class="mx-imgBorder"]
   > ![“权限”下拉菜单](../../_static/images/access-control/permission-drop-down.png)

2. 授予权限。 帐户中的所有用户都属于“所有用户”组。 管理员属于“管理员”组，该组对所有项目具有“管理”权限。

   若要向用户或组授予权限，请从“添加用户和组”下拉菜单中选择权限，然后单击“添加”： 

   > [!div class="mx-imgBorder"]
   > ![添加用户](../../_static/images/access-control/add-users.png)

   若要更改用户或组的权限，请从权限下拉菜单中选择新权限：

   > [!div class="mx-imgBorder"]
   > ![更改权限](../../_static/images/access-control/change-permissions.png)

3. 单击“保存更改”保存所做的更改，或单击“取消”放弃所做的更改。

### <a name="mlflow-artifact-permissions"></a>MLflow 项目权限

每个 [MLflow 试验](../../applications/mlflow/tracking.md#experiments)都有一个“项目位置”，用于存储记录到 MLflow 运行的项目。 从 MLflow 1.11 开始，项目默认会存储在 [Databricks 文件系统 (DBFS)](../../data/databricks-file-system.md) 的由 MLflow 管理的子目录中。
[MLflow 试验权限](#mlflow-experiment-permissions)适用于存储在这些托管位置中的项目，其前缀为 `dbfs:/databricks/mlflow-tracking`。 若要下载或记录项目，必须对其关联的 MLflow 试验具有相应级别的访问权限。

> [!NOTE]
>
> * 只能使用 MLflow 客户端（`1.9.1` 或更高版本）访问存储在 MLflow 所管理的位置中的项目。该 MLflow 客户端适用于 [Python](https://pypi.org/project/mlflow/)、[Java](https://mvnrepository.com/artifact/org.mlflow/mlflow-client) 和 [R](https://cran.r-project.org/web/packages/mlflow/index.html)。MLflow 管理的位置不支持其他访问机制，例如 [dbutils](../../dev-tools/databricks-utils.md#dbutils-fs) 和 [DBFS API](../../dev-tools/api/latest/dbfs.md)。
> * 创建 MLflow 试验时，还可以指定自己的项目位置。 对于存储在默认的由 MLflow 管理的 DBFS 目录之外的项目，不会强制执行试验访问控制。

## <a name="mlflow-model-permissions"></a>MLflow 模型权限

可以为在 [MLflow 模型注册表](../../applications/mlflow/model-registry.md)中注册的 [MLflow 模型](../../applications/mlflow/models.md)分配六个权限级别：“无权限”、“读取”、“编辑”、“管理过渡版本”、“管理生产版本”和“管理”。 该表列出了每个权限赋予用户的能力。

> [!NOTE]
>
> 模型版本从其父模型继承权限；不能设置模型版本的权限。

| 能力                                                                                         | 无权限   | 读取     | 编辑    | 管理过渡版本                 | 管理生产版本   | 管理    |
|-------------------------------------------------------------------------------------------------|------------------|----------|---------|-----------------------------------------|------------------------------|-----------|
| 创建模型                                                                                  | x                | x        | x       | x                                       | x                            | x         |
| 查看模型详细信息、版本、阶段转换请求、活动以及项目下载 URI |                  | x        | x       | x                                       | x                            | x         |
| 请求模型版本阶段转换                                                        |                  | x        | x       | x                                       | x                            | x         |
| 向模型添加版本                                                                        |                  |          | x       | x                                       | x                            | x         |
| 更新模型和版本说明                                                            |                  |          | x       | x                                       | x                            | x         |
| 在阶段之间转换模型版本                                                         |                  |          |         | x（在“无”、“已存档”和“正在过渡”之间） | x                            | x         |
| 批准或拒绝模型版本阶段转换请求                                      |                  |          |         | x（在“无”、“已存档”和“暂存”之间） | x                            | x         |
| 取消模型版本阶段转换请求（请参阅[注意](#transition-note)）                  |                  |          |         |                                         |                              | x         |
| 修改权限                                                                              |                  |          |         |                                         |                              | x         |
| 重命名模型                                                                                    |                  |          |         |                                         |                              | x         |
| 删除模型和模型版本                                                                 |                  |          |         |                                         |                              | x         |

> [!NOTE]
>
> 阶段转换请求的创建者也可以取消请求。

### <a name="default-mlflow-model-permissions"></a><a id="default-mlflow-model-permissions"> </a><a id="transition-note"> </a>默认的 MLflow 模型权限

* 以下权限独立于工作区对象访问控制而存在：
  * 所有用户都有权新建一个经过注册的模型。
  * 所有管理员对所有模型都具有“管理”权限。
* 在禁用工作区对象访问控制的情况下，存在以下权限：
  * 所有用户对所有模型都具有“管理”权限。
* 在[启用了工作区对象访问控制](../../administration-guide/access-control/workspace-acl.md#enable-workspace-acl)的情况下，存在以下默认权限：
  * 所有用户都对自己创建的模型有“管理”权限。
  * 非管理员用户对并非自己创建的模型具有的权限为“无权限”。

### <a name="configure-mlflow-model-permissions"></a>配置 MLflow 模型权限

你的帐户中的所有用户都属于“`all users`”组。 管理员属于“`admins`”组，该组对所有对象具有“管理”权限。

> [!NOTE]
>
> 此部分介绍如何使用 UI 来管理权限。 你还可以使用[权限 API](../../_static/api-refs/permissions-azure.yaml)。

1. 单击边栏中的 ![“模型”图标](../../_static/images/mlflow/models-icon.png) 图标。
2. 单击模型名称。
3. 单击模型名称右侧的![下拉按钮](../../_static/images/button-down.png)，然后选择“权限”。

   > [!div class="mx-imgBorder"]
   > ![“权限”下拉菜单](../../_static/images/access-control/model-permission.png)

4. 单击“选择用户或组”下拉菜单，选择某个用户或组。

   > [!div class="mx-imgBorder"]
   > ![添加用户](../../_static/images/access-control/select-user.png)

5. 选择权限。 若要更改用户或组的权限，请从权限下拉菜单中选择新权限：

   > [!div class="mx-imgBorder"]
   > ![更改权限](../../_static/images/access-control/select-permission.png)

6. 单击“添加”。
7. 单击“保存”保存所做的更改，或单击“取消”放弃所做的更改。

### <a name="mlflow-model-artifact-permissions"></a>MLflow 模型项目权限

每个 [MLflow 模型版本](../../applications/mlflow/model-registry.md#model-registry-concepts)的模型文件都存储在一个由 MLflow 管理的位置，其前缀为 `dbfs:/databricks/model-registry/`。

若要获取模型版本的文件的确切位置，你必须对模型具有“读取”访问权限。 使用 [REST API](../../dev-tools/api/latest/mlflow.md) 终结点 `/api/2.0/mlflow/model-versions/get-download-uri`。 获取 URI 后，可以使用 [DBFS API](../../dev-tools/api/latest/dbfs.md) 来下载文件。

MLflow 客户端（适用于 [Python](https://pypi.org/project/mlflow/)、[Java](https://mvnrepository.com/artifact/org.mlflow/mlflow-client) 和 [R](https://cran.r-project.org/web/packages/mlflow/index.html)）提供了几个简便方法，这些方法可以包装此工作流以下载和加载模型，例如 `mlflow.<flavor>.load_model()`。

> [!NOTE]
>
> MLflow 管理的文件位置不支持其他访问机制，例如 [dbutils](../../dev-tools/databricks-utils.md#dbutils-fs) 和 `%fs`。

## <a name="library-and-jobs-access-control"></a>库和作业访问控制

![库](../../_static/images/access-control/library.png) 所有用户均可查看库。 若要控制谁可以将库附加到群集，请参阅[群集访问控制](cluster-acl.md)。

![作业](../../_static/images/access-control/jobs.png) 若要控制谁可以运行作业并查看作业运行结果，请参阅[作业访问控制](jobs-acl.md)。