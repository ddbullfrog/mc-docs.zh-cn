---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: Azure Databricks 上的 RStudio - Azure Databricks
description: 了解如何将 Azure Databricks 上的 RStudio 与 R 配合使用。
ms.openlocfilehash: b2b26713e5298ec79240bd892fa32d1a5286cc17
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473065"
---
# <a name="rstudio-on-azure-databricks"></a>Azure Databricks 上的 RStudio

Azure Databricks 集成了 [RStudio Server](https://www.rstudio.com/products/rstudio/)，后者是适用于 R 的常用集成开发环境 (IDE)。

你可以使用 Azure Databricks 上的 RStudio Server 的开源版或专业版。 如果要使用 RStudio Server 专业版，则必须将现有的 RStudio 专业版许可证传输到 Azure Databricks（请参阅 [RStudio Server 专业版入门](#rsp)）。

用于机器学习的 Databricks Runtime 包括 RStudio Server 1.2 版开源包的未修改版本，可以在 [GitHub](https://github.com/rstudio/rstudio/tree/v1.2) 中找到该版本的源代码。 Databricks Runtime ML 不支持 RStudio Server 1.3 版。

## <a name="rstudio-integration-architecture"></a><a id="rsf"> </a><a id="rstudio-integration-architecture"> </a>RStudio 集成体系结构

使用 Azure Databricks 上的 RStudio Server 时，RStudio Server 守护程序在 Azure Databricks 群集的驱动程序节点（或主节点）上运行。 RStudio Web UI 通过 Azure Databricks Webapp 进行代理，这意味着你无需对群集网络配置进行任何更改。 下图演示了 RStudio 集成组件体系结构。

> [!div class="mx-imgBorder"]
> ![Databricks 上的 RStudio 的体系结构](../../../_static/images/clusters/rstudio-architecture.png)

> [!WARNING]
>
> Azure Databricks 通过群集的 Spark 驱动程序上的端口 8787 充当 RStudio Web 服务的代理。 此 Web 代理仅与 RStudio 配合使用。 如果你在端口 8787 上启动其他 Web 服务，用户可能会受到安全攻击。
> Databricks 和 Microsoft 均不负责因在群集上安装不受支持的软件而导致的任何问题。

## <a name="requirements"></a>要求

* 群集不得启用[表访问控制](../../../security/access-control/table-acls/object-privileges.md)或[自动终止](../../../clusters/clusters-manage.md#automatic-termination)。
* 你必须拥有对该群集的“可附加到”权限。 群集管理员可以向你授予此权限。 请参阅[群集访问控制](../../../security/access-control/cluster-acl.md)。
* 如果要使用专业版，则需要具有 RStudio Server 浮动专业版许可证。

## <a name="get-started-with-rstudio-server-open-source"></a>开始使用 RStudio Server 开源版

> [!IMPORTANT]
>
> Databricks Runtime 7.0 ML 上安装了 RStudio Server 开源版。 如果你使用的是 Databricks Runtime 7.0 ML 或更高版本，则可以跳过安装 RStudio Server 的部分。

若要开始使用 Azure Databricks 上的 RStudio Server 开源版，必须在 Azure Databricks 群集上安装 RStudio。 此安装只需执行一次。 安装通常由管理员执行。

### <a name="install-rstudio-server-open-source"></a>安装 RStudio Server 开源版

若要在 Azure Databricks 群集上安装 RStudio Server 开源版，你必须创建一个初始化脚本来安装 RStudio Server 开源版二进制程序包。 有关更多详细信息，请参阅[以群集为作用域的初始化脚本](../../../clusters/init-scripts.md#cluster-scoped-init-script)。  下面是一个笔记本单元示例，它在 DBFS 上的某个位置安装初始化脚本。

```python
script = """#!/bin/bash

set -euxo pipefail
RSTUDIO_BIN="/usr/sbin/rstudio-server"

if [[ ! -f "$RSTUDIO_BIN" && $DB_IS_DRIVER = "TRUE" ]]; then
  apt-get update
  apt-get install -y gdebi-core
  cd /tmp
  # You can find new releases at https://rstudio.com/products/rstudio/download-server/debian-ubuntu/.
  wget https://download2.rstudio.org/server/trusty/amd64/rstudio-server-1.2.5001-amd64.deb
  sudo gdebi -n rstudio-server-1.2.5001-amd64.deb
  rstudio-server restart || true
fi
"""

dbutils.fs.mkdirs("/databricks/rstudio")
dbutils.fs.put("/databricks/rstudio/rstudio-install.sh", script, True)
```

1. 在一个笔记本中运行此代码以安装 `dbfs:/databricks/rstudio/rstudio-install.sh` 处的脚本
2. 在启动群集之前，请将 `dbfs:/databricks/rstudio/rstudio-install.sh` 添加为初始化脚本。 有关详细信息，请参阅[诊断日志](../../../clusters/init-scripts.md#cluster-scoped-init-script)。
3. 启动群集。

### <a name="use-rstudio-server-open-source"></a>使用 RStudio Server 开源版

1. 显示你在其中安装了 RStudio 的群集的详细信息，并单击“应用”选项卡：

   > [!div class="mx-imgBorder"]
   > ![群集“应用”选项卡](../../../_static/images/clusters/rstudio-apps-ui.png)

2. 在“应用”选项卡中，单击“设置 RStudio”按钮。 这会为你生成一次性密码。
   单击“显示”链接以显示它并复制密码。

   > [!div class="mx-imgBorder"]
   > ![RStudio 一次性密码](../../../_static/images/clusters/rstudio-password-ui.png)

3. 单击“打开 RStudio UI”链接，在新选项卡中打开 UI。在登录窗体中以输入用户名和密码的方式登录。

   > [!div class="mx-imgBorder"]
   > ![RStudio 登录窗体](../../../_static/images/clusters/rstudio-login-ui.png)

4. 在 RStudio UI 中，你可以导入 SparkR 包并设置一个 SparkR 会话，以便在群集上启动 Spark 作业。

   ```r
   library(SparkR)
   sparkR.session()
   ```

   > [!div class="mx-imgBorder"]
   > ![RStudio 会话](../../../_static/images/clusters/rstudio-session-ui.png)

5. 你还可以附加 [sparklyr](sparklyr.md) 包并设置 Spark 连接。

   ```r
   SparkR::sparkR.session()
   library(sparklyr)
   sc <- spark_connect(method = "databricks")
   ```

   > [!div class="mx-imgBorder"]
   > ![包含 sparklyr 的 RStudio 会话](../../../_static/images/clusters/rstudio-session-ui-sparklyr.png)

## <a name="get-started-with-rstudio-server-pro"></a><a id="get-started-with-rstudio-server-pro"> </a><a id="rsp"> </a>开始使用 RStudio Server 专业版

### <a name="set-up-rstudio-license-server"></a>设置 RStudio 许可证服务器

若要在 Azure Databricks 上使用 RStudio Server 专业版，你需要将专业版许可证转换为[浮动许可证](https://support.rstudio.com/hc/articles/115011574507-Floating-Licenses)。
如需帮助，请联系 [help@rstudio.com](mailto:help@rstudio.com)。
当你的许可证已转换时，你必须为 RStudio Server 专业版设置[许可证服务器](https://www.rstudio.com/floating-license-servers/)。

若要设置许可证服务器，请执行以下操作：

1. 在你的云提供商网络上启动一个小型实例；许可证服务器守护程序是很轻型的程序。
2. 在你的实例上下载并安装相应版本的 RStudio 许可证服务器，然后启动该服务。 有关详细说明，请参阅 [RStudio Server 专业版文档](https://docs.rstudio.com/ide/server-pro/license-management.html#floating-licensing)。
3. 请确保通向 Azure Databricks 实例的许可证服务器端口已打开。

### <a name="install-rstudio-server-pro"></a>安装 RStudio Server 专业版

若要在 Azure Databricks 群集上安装 RStudio Server 专业版，你必须创建一个初始化脚本来安装 RStudio Server 专业版二进制程序包，并将其配置为使用你的许可证服务器进行许可证租用。
有关更多详细信息，请参阅[诊断日志](../../../clusters/init-scripts.md#cluster-scoped-init-script)。

> [!NOTE]
>
> 如果你计划在已包含 RStudio Server 开源版程序包的 Databricks Runtime 版本上安装 RStudio Server 专业版，则需要首先卸载该程序包，这样才能安装成功。

下面是一个笔记本单元示例，它在 DBFS 上生成初始化脚本。 该脚本还会执行其他身份验证配置，以简化与 Azure Databricks 的集成。

```python
script = """#!/bin/bash

set -euxo pipefail

if [[ $DB_IS_DRIVER = "TRUE" ]]; then
  sudo apt-get update
  sudo dpkg --purge rstudio-server # in case open source version is installed.
  sudo apt-get install -y gdebi-core alien

  ## Installing RStudio Server Pro
  cd /tmp

  # You can find new releases at https://rstudio.com/products/rstudio/download-commercial/debian-ubuntu/.
  wget https://download2.rstudio.org/server/trusty/amd64/rstudio-server-pro-1.2.5001-3-amd64.deb
  sudo gdebi -n rstudio-server-pro-1.2.5001-3-amd64.deb

  ## Configuring authentication
  sudo echo 'auth-proxy=1' >> /etc/rstudio/rserver.conf
  sudo echo 'auth-proxy-user-header-rewrite=^(.*)$ $1' >> /etc/rstudio/rserver.conf
  sudo echo 'auth-proxy-sign-in-url=<domain>/login.html' >> /etc/rstudio/rserver.conf
  sudo echo 'admin-enabled=1' >> /etc/rstudio/rserver.conf
  sudo echo 'export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin' >> /etc/rstudio/rsession-profile

  # Enabling floating license
  sudo echo 'server-license-type=remote' >> /etc/rstudio/rserver.conf

  # Session configurations
  sudo echo 'session-rprofile-on-resume-default=1' >> /etc/rstudio/rsession.conf
  sudo echo 'allow-terminal-websockets=0' >> /etc/rstudio/rsession.conf

  sudo rstudio-server license-manager license-server <license-server-url>
  sudo rstudio-server restart || true
fi
"""

dbutils.fs.mkdirs("/databricks/rstudio")
dbutils.fs.put("/databricks/rstudio/rstudio-install.sh", script, True)
```

1. 将 `<domain>` 替换为你的 Azure Databricks URL，并将 `<license-server-url>` 替换为你的浮动许可证服务器的 URL。
2. 在一个笔记本中运行此代码以安装 `dbfs:/databricks/rstudio/rstudio-install.sh` 处的脚本
3. 在启动群集之前，请将 `dbfs:/databricks/rstudio/rstudio-install.sh` 添加为初始化脚本。 有关详细信息，请参阅[诊断日志](../../../clusters/init-scripts.md#cluster-scoped-init-script)。
4. 启动群集。

### <a name="use-rstudio-server-pro"></a>使用 RStudio Server 专业版

1. 显示你在其中安装了 RStudio 的群集的详细信息，并单击“应用”选项卡：

   > [!div class="mx-imgBorder"]
   > ![群集“应用”选项卡](../../../_static/images/clusters/rstudio-apps-ui.png)

2. 在“应用”选项卡中，单击“设置 RStudio”按钮。

   > [!div class="mx-imgBorder"]
   > ![RStudio 一次性密码](../../../_static/images/clusters/rstudio-password-ui.png)

3. 你不需要使用这个一次性密码。 单击“打开 RStudio UI”链接，它会为你打开一个经身份验证的 RStudio 专业版会话。
4. 在 RStudio UI 中，你可以附加 SparkR 包并设置一个 SparkR 会话，以便在群集上启动 Spark 作业。

   ```r
   library(SparkR)
   sparkR.session()
   ```

   > [!div class="mx-imgBorder"]
   > ![RStudio 会话](../../../_static/images/clusters/rstudio-pro-session-ui.png)

5. 你还可以附加 [sparklyr](sparklyr.md) 包并设置 Spark 连接。

   ```r
   SparkR::sparkR.session()
   library(sparklyr)
   sc <- spark_connect(method = "databricks")
   ```

   > [!div class="mx-imgBorder"]
   > ![包含 sparklyr 的 RStudio 会话](../../../_static/images/clusters/rstudio-pro-session-ui-sparklyr.png)

## <a name="frequently-asked-questions-faq"></a>常见问题 (FAQ)

**RStudio Server 开源版与 RStudio Server 专业版之间的区别是什么？**

RStudio Server 专业版支持各种企业功能，这些功能在开源版上不可用。 可以在 [RStudio Inc 网站](https://www.rstudio.com/products/rstudio/#Server)上查看功能比较情况。

此外，RStudio Server 开源版根据 [GNU Affero 通用公共许可证 (AGPL)](https://www.gnu.org/licenses/agpl-3.0.en.html) 分发，而专业版为不能使用 AGPL 软件的组织提供了商业许可证。

最后，RStudio Server 专业版享受 RStudio Inc. 提供的专业和企业支持，而 RStudio Server 开源版没有这些支持。

**是否可以在 Azure Databricks 上使用我的 RStudio Server 专业版许可证？**

可以。如果你已有 RStudio Server 的专业版或企业版许可证，则可以在 Azure Databricks 上使用该许可证。 若要了解如何在 Azure Databricks 上安装 RStudio Server 专业版，请参阅[开始使用 RStudio Server 专业版](#rsp)。

**RStudio Server 在何处运行？我是否需要管理任何其他服务/服务器？**

正如 [RStudio 集成体系结构](#rsf)中的关系图所示，RStudio Server 守护程序在 Azure Databricks 群集的驱动程序节点（主节点）上运行。 使用 RStudio Server 开源版，无需运行任何其他服务器/服务。 但是，对于 RStudio Server 专业版，你必须管理一个运行 RStudio 许可证服务器的单独实例。

**是否可以在标准群集上使用 RStudio Server？**

可以。 最初，你需要使用[高并发](../../../clusters/configure.md#high-concurrency)群集，但该限制已不再存在。

**我应该如何在 RStudio 上持久保存我的工作？**

我们强烈建议你使用 RStudio 中的版本控制系统来持久保存工作。 RStudio 对各种版本控制系统提供了很大的支持，允许你签入和管理你的项目。

你还可以在 [Databricks 文件系统 (DBFS)](../../../data/databricks-file-system.md) 上保存文件（代码或数据）。 例如，如果你将文件保存在 `/dbfs/` 下，则在群集终止或重启时不会删除这些文件。

> [!IMPORTANT]
>
> 如果你不通过版本控制或 DBFS 来持久保存代码，则在管理员重启或终止群集时，你可能会丢失工作。

另一种方法是将 R 笔记本作为 `Rmarkdown` 导出以将其保存到你的本地文件系统，然后将该文件导入到 RStudio 实例中。

[Sharing R Notebooks using RMarkdown](https://databricks.com/blog/2018/07/06/sharing-r-notebooks-using-rmarkdown.html)（使用 RMarkdown 共享 R 笔记本）这一博客文章更详细地介绍了这些步骤。

**如何启动 `SparkR` 会话？**

`SparkR` 包含在 Databricks Runtime 中，但你必须将其加载到 RStudio 中。 在 RStudio 中运行以下代码以初始化 `SparkR` 会话。

```r
library(SparkR)
sparkR.session()
```

如果导入 `SparkR` 包时出错，请运行 `.libPaths()` 并验证结果中是否包含 `/home/ubuntu/databricks/spark/R/lib`。

如果未包含此内容，请检查 `/usr/lib/R/etc/Rprofile.site` 的内容。
列出驱动程序上的 `/home/ubuntu/databricks/spark/R/lib/SparkR`，以验证是否安装了 `SparkR` 包。

**如何启动 `sparklyr` 会话？**

必须在群集上安装 `sparklyr` 包。 使用以下方法之一安装 `sparklyr` 包：

* 作为 Azure Databricks 库
* `install.packages()` 命令
* RStudio 包管理 UI

`SparkR` 包含在 Databricks Runtime 中，但你必须将其加载到 RStudio 中。 在 RStudio 中运行以下代码以初始化 `sparklyr` 会话。

```r
SparkR::sparkR.session()
library(sparklyr)
sc <- spark_connect(method = “databricks”)
```

如果 `sparklyr` 命令失败，请确认 `SparkR::sparkR.session()` 是否成功。

**RStudio 如何与 Azure Databricks R 笔记本集成？**

你可以通过版本控制在笔记本与 RStudio 之间移动你的工作。

**什么是工作目录？**

当你在 RStudio 中启动项目时，你选择了一个工作目录。 默认情况下，这是在其中运行 RStudio Server 的驱动程序容器（主容器）中的主目录。 如果需要，你可以更改此目录。

**是否可以从 Azure Databricks 上运行的 RStudio 启动 Shiny 应用？**

非常遗憾，Azure Databricks 尚不支持 Shiny 应用和 RStudio Connect 的集成。

**无法在 Azure Databricks 上的 RStudio 中使用终端/git。如何解决此问题？**

请确保已禁用 WebSocket。 在 RStudio Server 开源版中，你可以从 UI 执行此操作。

> [!div class="mx-imgBorder"]
> ![RStudio 会话](../../../_static/images/clusters/rstudio-terminal-options.png)

在 RStudio Server 专业版中，你可以将 `allow-terminal-websockets=0` 添加到 `/etc/rstudio/rsession.conf`，以便对所有用户禁用 Websocket。

**我在群集详细信息下看不到“应用”选项卡。**

此功能并非可供所有客户使用。 你必须已参加 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)。