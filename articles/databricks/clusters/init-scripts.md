---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/04/2020
title: 群集节点初始化脚本 - Azure Databricks
description: 了解如何使用初始化 (init) 脚本在 Azure Databricks 群集上安装包和库，设置系统属性和环境变量，修改 Apache Spark 配置参数以及设置其他配置。
ms.openlocfilehash: cf5734b4425c3f59adf3893fa27e6cfca660d1de
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121871"
---
# <a name="cluster-node-initialization-scripts"></a><a id="cluster-node-initialization-scripts"> </a><a id="init-scripts"> </a>群集节点初始化脚本

Init 脚本是一个 shell 脚本，它在 Apache Spark 驱动程序或辅助 JVM 启动之前，在每个群集节点启动期间运行。

Init 脚本执行的任务的一些示例包括：

* 安装 Databricks Runtime 中未包含的包和库。 若要安装 Python 包，请使用位于 `/databricks/python/bin/pip` 的 Azure Databricks `pip` 二进制文件，确保 Python 包安装到 Azure Databricks Python 虚拟环境中，而不是系统 Python 环境中。 例如，`/databricks/python/bin/pip install <package-name>`。
* 在[特殊情况](https://spark.apache.org/docs/latest/sql-programming-guide.html#troubleshooting)下，修改 JVM 系统类路径。
* 设置 JVM 使用的系统属性和环境变量。
* 修改 Spark 配置参数。

## <a name="init-script-types"></a>初始化脚本类型

Azure Databricks 支持两种 init 脚本：群集范围脚本和全局脚本。

* **群集范围脚本**：在使用脚本配置的每个群集上运行。 这是运行 init 脚本的推荐方法。
* **全局脚本**（公共预览版）：在工作区中的每个群集上运行。 这些脚本有助于在工作区中强制实施一致的群集配置。 请谨慎使用，因为它们可能会造成非预期影响，例如库冲突。 只有管理员用户才能创建全局 init 脚本。

  > [!NOTE]
  >
  > Azure Databricks 最近改进了全局 init 脚本的行为，以便以更安全、更可见和更安全的方式运行。 应将现有的旧版全局 init 脚本迁移到新版全局 init 脚本框架。 请参阅[从旧版本迁移到新版全局 init 脚本](#migrate-legacy-scripts)。

另外，还有两种不推荐使用的 init 脚本。 Databricks 建议你将这些类型的 init 脚本迁移到上面列出的脚本：

* **群集命名脚本**：在与脚本名称相同的群集上运行。 群集命名 init 脚本会尽力运行（以无提示方式忽略故障），并尝试继续执行群集启动过程。 应改为使用群集范围 init 脚本，作为完整的替代方式。
* **旧版全局脚本**：在每个群集上运行。 它们不如新版全局 init 脚本框架安全，会以无提示方式忽略故障，并且无法引用[环境变量](#env-var)。 应改用新版全局 init 脚本框架。

每次更改任何类型的 init 脚本时，都必须重启受该脚本影响的所有群集。

## <a name="init-script-execution-order"></a>初始化脚本执行顺序

Init 脚本的执行顺序如下：

1. 旧版全局脚本
2. 群集命名脚本
3. 全局脚本（新版）
4. 群集范围脚本

## <a name="environment-variables"></a><a id="env-var"> </a><a id="environment-variables"> </a>环境变量

群集范围和全局 init 脚本（新版）支持以下环境变量：

* `DB_CLUSTER_ID`：运行脚本的群集的 ID。 请参阅[群集 API](../dev-tools/api/latest/clusters.md)。
* `DB_CONTAINER_IP`：运行 Spark 的容器的专用 IP 地址。 Init 脚本在此容器内运行。 请参阅 [SparkNode](../dev-tools/api/latest/clusters.md#clustersparkinfosparknode)。
* `DB_IS_DRIVER`：脚本是否在驱动程序节点上运行。
* `DB_DRIVER_IP`：驱动程序节点的 IP 地址。
* `DB_INSTANCE_TYPE`：主机 VM 的实例类型。
* `DB_CLUSTER_NAME`：要在其上执行脚本的群集的名称。
* `DB_PYTHON_VERSION`：在群集上使用的 Python 版本。 请参阅 [Python 版本](configure.md#python-3)。
* `DB_IS_JOB_CLUSTER`：是否创建群集来运行作业。 请参阅[创建作业](../jobs.md#create-a-job)。
* `SPARKPASSWORD`：指向[机密](../security/secrets/secrets.md#spark-conf-env-var)的路径。

例如，如果只想在驱动程序节点上运行脚本的一部分，则可以编写如下脚本：

```bash
echo $DB_IS_DRIVER
if [[ $DB_IS_DRIVER = "TRUE" ]]; then
  <run this part only on driver>
else
  <run this part only on workers>
fi
<run this part on both driver and workers>
```

## <a name="logging"></a>日志记录

在群集事件日志中捕获 init 脚本的开始和结束事件。 在群集日志中捕获详细信息。 在帐户级别诊断日志中捕获全局 init 脚本的创建、编辑和删除事件。

### <a name="init-script-events"></a>Init 脚本事件

[群集事件日志](clusters-manage.md#event-log)捕获两个 init 脚本事件：`INIT_SCRIPTS_STARTED` 和 `INIT_SCRIPTS_FINISHED`，指示计划执行的脚本和已成功完成的脚本。 `INIT_SCRIPTS_FINISHED` 还捕获执行持续时间。

在日志事件详细信息中，由密钥 `"global"` 指示全局 init 脚本，而由密钥 `"cluster"` 指示群集范围 init 脚本。

> [!NOTE]
>
> 群集事件日志不记录每个群集节点的 init 脚本事件；仅选择一个节点来代表全部。

### <a name="init-script-logs"></a><a id="init-script-log"> </a><a id="init-script-logs"> </a>脚本日志

如果为群集配置了[群集日志传送](configure.md#cluster-log-delivery)，则日志将传送到 `/databricks/init_scripts`。 对于每个容器，日志将显示在名为 `init_scripts/<cluster_id>_<container_ip>` 的子目录中。 例如，如果将群集日志传送到 `dbfs:/cluster-logs`，则目录将为：`dbfs:/cluster-logs/init_scripts/<cluster_id>_<container_ip>`。 例如： 。

```bash
dbfs ls dbfs:/cluster-logs/1001-234039-abcde739/init_scripts
```

```console
1001-234039-abcde739_10_97_225_166
1001-234039-abcde739_10_97_231_88
1001-234039-abcde739_10_97_244_199
```

如果日志已传送到 DBFS，则可以使用[文件系统实用工具](../dev-tools/databricks-utils.md#dbutils-fs)查看日志。 否则，你可以在笔记本中使用以下代码来查看日志：

```bash
%sh
ls /databricks/init_scripts/
```

群集每次启动时，都会将日志写入 init 脚本日志文件夹。

> [!IMPORTANT]
>
> 创建群集并启用群集日志传送的任何用户都可以查看全局 init 脚本的 `stderr` 和 `stdout` 输出。 应确保全局 init 脚本不输出任何敏感信息。

### <a name="diagnostic-logs"></a><a id="cluster-scoped-init-script"> </a><a id="diagnostic-logs"> </a>诊断日志

Azure Databricks 诊断日志记录捕获事件类型为 `globalInitScripts` 的全局 init 脚本的创建、编辑和删除事件。 请参阅 [Azure Databricks 中的诊断日志记录](../administration-guide/account-settings/azure-diagnostic-logs.md)。

## <a name="cluster-scoped-init-scripts"></a>群集范围的初始化脚本

群集范围 init 脚本是在群集配置中定义的 init 脚本。 群集范围 init 脚本同时适用于你创建的群集和为运行作业而创建的群集。
由于脚本是群集配置的一部分，因此可通过[群集访问控制](../security/access-control/cluster-acl.md)来控制可以更改脚本的用户。

可使用 UI、CLI 以及调用 Clusters API 来配置群集范围的 init 脚本。 本部分重点介绍如何使用 UI 执行这些任务。 有关其他方法，请参阅 [Databricks CLI](../dev-tools/cli/index.md) 和[群集 API](../dev-tools/api/latest/clusters.md)。

你可以添加任意数量的脚本，这些脚本会按照所提供的顺序依次执行。

如果群集范围 init 脚本返回非零的退出代码，则群集启动会失败。 可通过配置[日志传送](configure.md#cluster-log-delivery)并检查[init 脚本日志](#init-script-log)来排查群集范围 init 脚本问题。

### <a name="cluster-scoped-init-script-locations"></a>群集范围 init 脚本位置

可以将 init 脚本放在群集可访问的 DBFS 目录中。 DBFS 中的群集节点 init 脚本必须存储在 [DBFS 根目录](../data/databricks-file-system.md#dbfs-root)中。 Azure Databricks 不支持将 init 脚本存储在通过[安装对象存储](../data/databricks-file-system.md#mount-storage)创建的 DBFS 目录中。

### <a name="example-cluster-scoped-init-scripts"></a><a id="example-cluster-scoped-init-scripts"> </a><a id="example-script"> </a>群集范围 init 脚本示例示例

本部分演示两个 init 脚本示例。

#### <a name="example-install-postgresql-jdbc-driver"></a>示例：安装 PostgreSQL JDBC 驱动程序

在 Python 笔记本中运行的以下代码片段可创建用于安装 PostgreSQL JDBC 驱动程序的 init 脚本。

1. 创建要在其中存储 init 脚本的 DBFS 目录。 本示例使用 `dbfs:/databricks/scripts`。

   ```python
   dbutils.fs.mkdirs("dbfs:/databricks/scripts/")
   ```

2. 在该目录中创建一个名为 `postgresql-install.sh` 的脚本：

   ```python
   dbutils.fs.put("/databricks/scripts/postgresql-install.sh","""
   #!/bin/bash
   wget --quiet -O /mnt/driver-daemon/jars/postgresql-42.2.2.jar https://repo1.maven.org/maven2/org/postgresql/postgresql/42.2.2/postgresql-42.2.2.jar""", True)
   ```

3. 检查该脚本是否存在。

   ```python
   display(dbutils.fs.ls("dbfs:/databricks/scripts/postgresql-install.sh"))
   ```

也可在本地创建 init 脚本 `postgresql-install.sh`：

```bash
#!/bin/bash
wget --quiet -O /mnt/driver-daemon/jars/postgresql-42.2.2.jar https://repo1.maven.org/maven2/org/postgresql/postgresql/42.2.2/postgresql-42.2.2.jar
```

然后使用 [DBFS CLI](../dev-tools/cli/dbfs-cli.md) 将其复制到 `dbfs:/databricks/scripts`：

```bash
dbfs cp postgresql-install.sh dbfs:/databricks/scripts/postgresql-install.sh
```

#### <a name="example-use-conda-to-install-python-libraries"></a>示例：使用 conda 安装 Python 库

在 Databricks Runtime ML 中，使用 [Conda](https://conda.io/docs/) 包管理器安装 Python 包。 若要在群集初始化时安装 Python 库，可以使用以下脚本：

```bash
#!/bin/bash
set -ex
/databricks/python/bin/python -V
. /databricks/conda/etc/profile.d/conda.sh
conda activate /databricks/python
conda install -y astropy
```

> [!NOTE]
>
> 有关在群集上安装 Python 包的其他方法，请参阅[库](../libraries/index.md)。

### <a name="configure-a-cluster-scoped-init-script"></a><a id="config-cluster-scoped"> </a><a id="configure-a-cluster-scoped-init-script"> </a>配置群集范围 init 脚本

可使用 UI 或 API 配置群集来运行 init 脚本。

> [!IMPORTANT]
>
> * 该脚本必须存在于配置的位置。 如果该脚本不存在，则群集将无法启动或自动纵向扩展。
> * Init 脚本不能大于 64KB。 如果脚本超过该大小，则群集将无法启动，并会在群集日志中显示失败消息。

#### <a name="configure-a-cluster-scoped-init-script-using-the-ui"></a>使用 UI 配置群集范围 init 脚本

使用群集配置页面配置群集以运行 init 脚本：

1. 在群集配置页面上，单击“高级选项”切换开关。
2. 在页面底部，单击“Init 脚本”选项卡。

   > [!div class="mx-imgBorder"]
   > ![“Init 脚本”选项卡](../_static/images/clusters/init-scripts-azure.png)

3. 在“目标”下拉列表中，选择一个目标类型。 在上一部分的示例中，目标为 `DBFS`。
4. 指定 init 脚本的路径。 在上一部分的示例中，路径为 `dbfs:/databricks/scripts/postgresql-install.sh`。 路径必须以 `dbfs:/` 开头。
5. 单击“添加” 。

若要从群集配置中删除脚本，请单击脚本右侧的 ![“删除”图标](../_static/images/clusters/delete-icon.png) 。 确认删除后，系统将提示你重启群集。 （可选）可根据需要从脚本上传到的位置删除脚本文件。

#### <a name="configure-a-cluster-scoped-init-script-using-the-dbfs-rest-api"></a>使用 DBFS REST API 配置群集范围 init 脚本

若要使用 [Clusters API](../dev-tools/api/latest/clusters.md) 配置 ID 为 `1202-211320-brick1` 的群集，使其运行上一部分中的 init 脚本，请运行以下命令：

```bash
curl -n -X POST -H 'Content-Type: application/json' -d '{
  "cluster_id": "1202-211320-brick1",
  "num_workers": 1,
  "spark_version": "6.4.x-scala2.11",
  "node_type_id": "Standard_D3_v2",
  "cluster_log_conf": {
    "dbfs" : {
      "destination": "dbfs:/cluster-logs"
    }
  },
  "init_scripts": [ {
    "dbfs": {
      "destination": "dbfs:/databricks/scripts/postgresql-install.sh"
    }
  } ]
}' https://<databricks-instance>/api/2.0/clusters/edit
```

## <a name="global-init-scripts-new"></a><a id="global-init-script"> </a><a id="global-init-scripts-new"> </a>全局 init 脚本（新版）

> [!IMPORTANT]
>
> 新版全局 init 脚本框架以[公共预览版](../release-notes/release-types.md)提供。

全局 init 脚本在工作区中创建的每个群集上运行。 如果希望强制执行组织范围的库配置或安全屏幕，建议使用全局 init 脚本。 只有管理员才能创建全局 init 脚本。 你可以使用 UI 或 REST API 来创建它们。

> [!IMPORTANT]
>
> 请谨慎使用全局 init 脚本。 可以轻松地添加库，或进行其他可能造成非预期影响的修改。 应尽可能使用群集范围 init 脚本。

可通过配置[日志传送](configure.md#cluster-log-delivery)并检查 [init 脚本日志](#init-script-log)来排查全局 init 脚本问题。

> [!IMPORTANT]
>
> 创建群集并启用群集日志传送的任何用户都可以查看全局 init 脚本的 `stderr` 和 `stdout` 输出。 应确保全局 init 脚本不输出任何敏感信息。

### <a name="add-a-global-init-script-using-the-ui"></a><a id="add-a-global-init-script-using-the-ui"> </a><a id="global-init-script-ui"> </a>使用 UI 添加全局 init 脚本

若要使用管理控制台配置全局 init 脚本，请执行以下操作：

1. 转到管理控制台，然后单击“全局 init 脚本”选项卡。

   > [!div class="mx-imgBorder"]
   > ![“全局 init 脚本”选项卡](../_static/images/clusters/global-init-scripts-tab.png)

2. 单击“+ 添加”按钮。
3. 通过在“脚本”字段中键入内容、粘贴或拖动文本文件，为脚本命名。

   > [!div class="mx-imgBorder"]
   > ![添加全局 init 脚本](../_static/images/clusters/global-init-add.png)

   > [!NOTE]
   >
   > Init 脚本不能大于 64KB。 如果脚本超过该大小，则尝试保存时将出现错误消息。

4. 如果为工作区配置了多个全局 init 脚本，请设置新脚本的运行顺序。
5. 若要在保存后为所有新群集和重启的群集启用脚本，请打开“启动”开关。

   > [!IMPORTANT]
   >
   > 必须重启正在运行的群集，才能使对全局 init 脚本的更改生效，包括对运行顺序、名称和启用状态的更改。

6. 单击“添加” 。

### <a name="edit-a-global-init-script-using-the-ui"></a>使用 UI 编辑全局 init 脚本

1. 转到管理控制台，然后单击“全局 init 脚本”选项卡。
2. 单击一个脚本。
3. 编辑该脚本。
4. 单击“确认”  。

### <a name="configure-a-global-init-script-using-the-api"></a><a id="configure-a-global-init-script-using-the-api"> </a><a id="global-init-script-api"> </a>使用 API 配置全局 init 脚本

管理员可以使用[全局 init 脚本 API](../_static/api-refs/global-init-scripts-azure.yaml) 在工作区中添加、删除、重新排序并获取有关全局 init 脚本的信息。

### <a name="migrate-from-legacy-to-new-global-init-scripts"></a><a id="migrate-from-legacy-to-new-global-init-scripts"> </a><a id="migrate-legacy-scripts"> </a>从旧版迁移到新版全局 init 脚本

如果你的 Azure Databricks 工作区是在 2020 年 8 月之前启动的，则你可能仍然使用的是旧版全局 init 脚本。 应将它们迁移到新版全局 init 脚本框架中，以利用新版脚本框架中包含的安全性、一致性和可见性功能。

1. 复制现有的旧版全局 init 脚本，然后使用 [UI](#global-init-script-ui) 或 [REST API](#global-init-script-api) 将它们添加到新版全局 init 脚本框架中。

   使它们保持禁用状态，直到完成下一步。

2. 禁用所有旧版全局 init 脚本。

   在管理员控制台中，转到“全局 init 脚本”标签，然后关闭“旧版全局 init 脚本”开关 。

   > [!div class="mx-imgBorder"]
   > ![禁用旧版全局 init 脚本](../_static/images/clusters/disable-legacy-global-init-scripts.png)

3. 启用新版全局 init 脚本。

   在“全局 init 脚本”选项卡上，为要启用的每个 init 脚本打开“启用”开关 。

   > [!div class="mx-imgBorder"]
   > ![启动全局脚本](../_static/images/clusters/enable-global-scripts.png)

4. 重启所有群集。

   <!--important! Once legacy global init scripts are disabled, legacy scripts stop running immediately. This has two important impacts:-->
   * 对于在自动纵向扩展运行的群集期间添加的新节点，旧版脚本将不会在这些节点上运行。 新版全局 init 脚本也不会在这些新节点上运行。 如果全局脚本根本没有在新节点上运行，必须重启所有群集，才能确保新版脚本在这些新节点上运行，并且没有现有群集尝试添加新节点。
   * 迁移到新版全局 init 脚本框架并禁用旧脚本时，可能需要修改非幂等脚本。

## <a name="legacy-global-init-scripts-deprecated"></a><a id="legacy-global-init-script"> </a><a id="legacy-global-init-scripts-deprecated"> </a>旧版全局 init 脚本（已弃用）

全局 init 脚本在工作区中创建的每个群集上运行。

> [!IMPORTANT]
>
> 已弃用全局 init 脚本，而推荐使用新版全局 init 脚本框架，该框架更安全，可以查看故障并可以引用与群集相关的[环境变量](#env-var)。 应[将现有的旧版全局 init 脚本迁移到新框架](#migrate-legacy-scripts)，以利用这些改进。

旧版全局 init 脚本必须存储在 `dbfs:/databricks/init/`中。

> [!IMPORTANT]
>
> * 请谨慎使用全局 init 脚本。 可以轻松地添加库，或进行其他可能造成非预期影响的修改。 应尽可能使用群集范围 init 脚本。
> * 如果有多个旧版全局 init 脚本，则执行顺序不确定，并且取决于 DBFS 客户端返回脚本的顺序。

若要删除旧版全局 init 脚本，请删除 init 脚本文件。 可以使用 [DBFS API](../dev-tools/api/latest/dbfs.md) 或 [DBFS CLI](../dev-tools/cli/dbfs-cli.md) 在笔记本中执行此操作。 例如： 。

```python
dbutils.fs.rm("/databricks/init/my-echo.sh")
```

如果创建的旧版全局 init 脚本阻止了新群集的启动，请使用 API 或 CLI 来移动或删除该脚本。

### <a name="example-legacy-global-init-script"></a>示例旧版全局 init 脚本

在 Python 笔记本中运行的以下代码片段在 DBFS 位置 `/databricks/init/` 中创建了名为 `my-echo.sh` 的旧版全局 init 脚本：

1. 如果 `dbfs:/databricks/init/` 不存在，请进行创建。

   ```python
   dbutils.fs.mkdirs("dbfs:/databricks/init/")
   ```

2. 显示现有的全局 init 脚本的列表。

   ```python
   display(dbutils.fs.ls("dbfs:/databricks/init/"))
   ```

3. 创建仅追加到文件的脚本。

   ```python
   dbutils.fs.put("dbfs:/databricks/init/my-echo.sh" ,"""
   #!/bin/bash

   echo "hello" >> /hello.txt
   """, True)
   ```

4. 检查该脚本是否存在。

   ```python
   display(dbutils.fs.ls("dbfs:/databricks/init/"))
   ```

## <a name="cluster-named-init-scripts-deprecated"></a><a id="cluster-named-init-script"> </a><a id="cluster-named-init-scripts-deprecated"> </a>群集命名 init 脚本（已弃用）

群集命名脚本的作用域是单个群集，由群集名称指定。 群集命名 init 脚本必须存储在目录 `dbfs:/databricks/init/<cluster-name>` 中。 例如，若要为名为 `PostgreSQL` 的群集指定 init 脚本，请创建目录 `dbfs:/databricks/init/PostgreSQL`，然后将应在群集 `PostgreSQL` 上运行的所有脚本都放在该目录中。

> [!IMPORTANT]
>
> * 已弃用群集命名 init 脚本。 应使用[群集范围 init 脚本](#cluster-scoped-init-script)。
> * 不能对运行作业的群集使用群集命名 init 脚本，因为作业群集名称是动态生成的。 但可以将[群集范围](#cluster-scoped-init-script) init 脚本用于作业群集。
> * 请避免在群集名称中使用空格，因为它们用于脚本和输出路径中。
> * 如果有多个群集命名 init 脚本，则执行顺序不确定，并且取决于 DBFS 客户端返回脚本的顺序。

若要删除群集命名 init 脚本，请删除 init 脚本文件。 可使用 [DBFS API](../dev-tools/api/latest/dbfs.md) 或 [DBFS CLI](../dev-tools/cli/dbfs-cli.md) 在笔记本中执行此操作。 例如： 。

```python
dbutils.fs.rm("dbfs:/databricks/init/PostgreSQL/postgresql-install.sh")
```

### <a name="example-cluster-named-init-script"></a>群集命名 init 脚本示例

在 Python 笔记本中运行的以下代码片段可在 DBFS 位置 `/databricks/init` 中创建一个名为 `PostgreSQL` 的 init 脚本，该脚本用于在该群集上安装 PostgreSQL JDBC 驱动程序。 如果创建包含群集名称的变量 `clusterName`，则可以创建可自定义的命令。

1. 如果 `dbfs:/databricks/init/` 不存在，请进行创建。

   ```python
   dbutils.fs.mkdirs("dbfs:/databricks/init/")
   ```

2. 配置群集名称变量。

   ```python
   clusterName = "PostgreSQL"
   ```

3. 创建名为 `PostgreSQL` 的目录。

   ```python
   dbutils.fs.mkdirs("dbfs:/databricks/init/%s/"%clusterName)
   ```

4. 在目录 `PostgreSQL` 中创建脚本。

   ```python
   dbutils.fs.put("/databricks/init/PostgreSQL/postgresql-install.sh","""
   #!/bin/bash
   wget --quiet -O /mnt/driver-daemon/jars/postgresql-42.2.2.jar https://repo1.maven.org/maven2/org/postgresql/postgresql/42.2.2/postgresql-42.2.2.jar
   wget --quiet -O /mnt/jars/driver-daemon/postgresql-42.2.2.jar https://repo1.maven.org/maven2/org/postgresql/postgresql/42.2.2/postgresql-42.2.2.jar""", True)
   ```

5. 检查特定于群集的 init 脚本是否存在。

   ```python
   display(dbutils.fs.ls("dbfs:/databricks/init/%s/postgresql-install.sh"%clusterName))
   ```

## <a name="legacy-global-and-cluster-named-init-script-logs-deprecated"></a>旧版全局和群集命名的初始化脚本日志（已弃用）

Databricks 将旧版全局和群集命名 init 脚本的所有 init 脚本输出保存到 DBFS 中的文件，命名方式如下：`dbfs:/databricks/init/output/<cluster-name>/<date-timestamp>/<script-name>_<node-ip>.log`。 例如，如果群集 `PostgreSQL` 具有两个 IP 地址为 `10.0.0.1` 和 `10.0.0.2` 的 Spark 节点，并且 init 脚本目录中包含名为 `installpostgres.sh` 的脚本，则在以下路径中将有两个输出文件：

* `dbfs:/databricks/init/output/PostgreSQL/2016-01-01_12-00-00/installpostgres.sh_10.0.0.1.log`
* `dbfs:/databricks/init/output/PostgreSQL/2016-01-01_12-00-00/installpostgres.sh_10.0.0.2.log`