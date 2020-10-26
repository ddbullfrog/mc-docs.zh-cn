---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/01/2020
title: 使用 Databricks 容器服务自定义容器 - Azure Databricks
description: 了解如何使用自定义 Docker 映像创建 Azure Databricks 群集，以完全控制库自定义、环境锁定和 CI/CD 集成。
ms.openlocfilehash: 4d2d734993c598188c0e1c1be37bfeecf857038c
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121809"
---
# <a name="customize-containers-with-databricks-container-services"></a><a id="containers"> </a><a id="customize-containers-with-databricks-container-services"> </a>使用 Databricks 容器服务自定义容器

Databricks 容器服务允许你在创建群集时指定 Docker 映像。 一些示例用例包括：

* 库自定义 - 你可以完全控制你要安装的系统库。
* 黄金容器环境 - 你的 Docker 映像是锁定的环境，永远不会更改。
* Docker CI/CD 集成 - 可以将 Azure Databricks 与 Docker CI/CD 管道集成。

你还可以使用 Docker 映像在具有 GPU 设备的群集上创建自定义深度学习环境。 若要进一步了解如何将 GPU 群集与 Databricks 容器服务配合使用，请参阅 [GPU 群集上的 Databricks 容器服务](gpu.md#databricks-container-services-on-gpu-clusters)。

对于容器每次启动时要执行的任务，请使用[初始化脚本](#containers-init-script)。

## <a name="requirements"></a>要求

> [!NOTE]
>
> 用于机器学习的 Databricks Runtime 和用于基因组学的 Databricks Runtime 不支持 Databricks 容器服务。

* Databricks Runtime 6.1 或更高版本。 如果你以前使用过 Databricks 容器服务，则必须升级基础映像。 请参阅 [https://github.com/databricks/containers](https://github.com/databricks/containers) 中用 `6.x` 标记的最新映像。
* 你的 Azure Databricks 工作区必须已[启用](../administration-guide/clusters/container-services.md) Databricks 容器服务。
* 你的计算机必须运行最新的 Docker 守护程序（一个经过测试的可与客户端/服务器版本 18.03.0-ce 一起使用的版本），并且 `docker` 命令必须在你的 `PATH` 上可用。

## <a name="step-1-build-your-base"></a>步骤 1：生成基础映像

若要成功启动群集，必须满足 Azure Databricks 的几个最低要求。 因此，建议你根据 Azure Databricks 已生成并测试的基础映像生成 Docker 基础映像：

```bash
FROM databricksruntime/standard:latest
...
```

若要指定其他 Python 库（例如最新版本的 pandas 和 urllib），请使用特定于容器的 `pip` 版本。 对于 `datatabricksruntime/standard:latest` 容器，请包括以下内容：

```bash
RUN /databricks/conda/envs/dcs-minimal/bin/pip install pandas
RUN /databricks/conda/envs/dcs-minimal/bin/pip install urllib3
```

示例基础映像承载在 [https://hub.docker.com/u/databricksruntime](https://hub.docker.com/u/databricksruntime) 处的 Docker Hub 上。 用来生成这些基础映像的 Dockerfile 位于 [https://github.com/databricks/containers](https://github.com/databricks/containers)。

> [!NOTE]
>
> 基础映像 `databricksruntime/standard` 和 `databricksruntime/minimal` 不会与不相关的 `databricks-standard` 和 `databricks-minimal` 环境混淆，这些环境包含在不再可用的带有 Conda 的 Databricks Runtime（Beta 版本）中。

你还可以从头开始生成 Docker 基础映像。 你的 Docker 映像必须满足以下要求：

* 系统 `PATH` 上的 Java 为 JDK 8u191
* bash
* iproute2 ([ubuntu iproute](https://packages.ubuntu.com/search?keywords=iproute2))
* coreutils（[ubuntu coreutils](https://packages.ubuntu.com/search?keywords=coreutils)、[alpine coreutils](https://pkgs.alpinelinux.org/package/v3.3/main/x86/coreutils)）
* procps（[ubuntu procps](https://packages.ubuntu.com/search?keywords=procps)、[alpine procps](https://pkgs.alpinelinux.org/package/v3.3/main/x86/procps)）
* sudo（[ubuntu sudo](https://packages.ubuntu.com/search?keywords=sudo)、[alpine sudo](https://pkgs.alpinelinux.org/package/v3.3/main/x86/sudo)）
* Ubuntu 或 Alpine Linux

或者，你可以使用 `databricksruntime/minimal` 处由 Databricks 生成的最小映像。

当然，上面列出的最低要求不包括 Python、R、Ganglia 以及 Azure Databricks 群集中通常应该提供的许多其他功能。 若要获得这些功能，可以生成合适的基础映像（即 `databricksruntime/rbase`，适用于 R），或者参考 GitHub 中的 Dockerfile 来确定如何进行生成以支持你需要的特定功能。

> [!WARNING]
>
> 现在，你可以控制群集的环境了。 强大的功能意味着重大的责任。 灵活性太大容易造成破坏。 本文档根据我们的经验提供了一些建议。 最终，你将开始走出已知的领域，事情可能会变坏！ 与任何 Docker 工作流一样，第一次或第二次可能无法正常运行，但一旦开始正常运行，就始终可以正常运行。

## <a name="step-2-push-your-base-image"></a>步骤 2：推送基础映像

将自定义基础映像推送到 Docker 注册表。 此过程已通过 [Docker Hub](https://hub.docker.com/) 和 [Azure 容器注册表 (ACR)](/container-registry/) 进行了测试。 支持无身份验证或基本身份验证的 Docker 注册表应当能正常工作。

## <a name="step-3-launch-your-cluster"></a>步骤 3：启动群集

你可以使用 UI 或 API 启动群集。

### <a name="launch-your-cluster-using-the-ui"></a>使用 UI 启动群集

1. 指定支持 Databricks 容器服务的 Databricks Runtime 版本。

   > [!div class="mx-imgBorder"]
   > ![选择 Databricks 运行时](../_static/images/clusters/custom-container-azure.png)

2. 选择“使用自己的 Docker 容器”。
3. 在“Docker 映像 URL”字段中，输入你的自定义 Docker 映像。

   Docker 映像 URL 示例：

   * DockerHub：`<organization>/<repository>:<tag>`，例如：`databricksruntime/standard:latest`
   * Azure 容器注册表：`<your-registry-name>.azurecr.io/<repository-name>:<tag>`
4. 选择身份验证类型。

### <a name="launch-your-cluster-using-the-api"></a>使用 API 启动群集

1. [生成 API 令牌](../dev-tools/api/latest/authentication.md)。
2. 使用[群集 API](../dev-tools/api/latest/clusters.md) 启动包含自定义 Docker 基础映像的群集。

   ```bash
   curl -X POST -H "Authorization: Bearer <token>" https://<databricks-instance>/api/2.0/clusters/create -d '{
     "cluster_name": "<cluster-name>",
     "num_workers": 0,
     "node_type_id": "Standard_DS3_v2",
     "docker_image": {
       "url": "databricksruntime/standard:latest",
       "basic_auth": {
         "username": "<docker-registry-username>",
         "password": "<docker-registry-password>"
       }
     },
     "spark_version": "5.5.x-scala2.11",
   }'
   ```

   `basic_auth` 要求取决于你的 Docker 映像类型：

   * 对于公共 Docker 映像，不要包括 `basic_auth` 字段。
   * 对于专用 Docker 映像，必须包括 `basic_auth` 字段，使用服务主体 ID 和密码作为用户名和密码。
   * 对于 Azure ACR，必须包括 `basic_auth` 字段，使用服务主体 ID 和密码作为用户名和密码。 请参阅 [Azure ACR 服务主体身份验证文档](/container-registry/container-registry-auth-service-principal)，了解如何创建服务主体。

## <a name="use-an-init-script"></a><a id="containers-init-script"> </a><a id="use-an-init-script"> </a>使用初始化脚本

Databricks 容器服务群集允许客户在 Docker 容器中包括初始化脚本。 在大多数情况下，应避免使用初始化脚本，而应直接通过 Docker（使用 Dockerfile）进行自定义。 但是，某些任务必须在容器启动时执行，而不是在构建容器时执行。 请为这些任务使用初始化脚本。

例如，假设要在自定义容器中运行安全守护程序。 通过映像生成管道在 Docker 映像中安装并生成守护程序。 然后，添加启动守护程序的初始化脚本。 在此示例中，初始化脚本将包含一个类似于 `systemctl start my-daemon` 的行。

在 API 中，你可以将初始化脚本指定为群集规范的一部分，如下所示。 有关详细信息，请参阅 [InitScriptInfo](../dev-tools/api/latest/clusters.md#initscriptinfo)。

```bash
"init_scripts": [
    {
        "file": {
            "destination": "file:/my/local/file.sh"
        }
    }
]
```

对于 Databricks 容器服务映像，你还可以将初始化脚本存储在 DBFS 或云存储中。

启动 Databricks 容器服务群集时，将执行以下步骤：

1. 从云提供商处获取 VM。
2. 从你的存储库中下载自定义 Docker 映像。
3. Azure Databricks 基于映像创建 Docker 容器。
4. Databricks Runtime 代码复制到 Docker 容器中。
5. 执行初始化脚本。 请参阅[初始化脚本执行顺序](init-scripts.md#init-script-execution-order)。

Azure Databricks 会忽略 Docker `CMD` 和 `ENTRYPOINT` 基元。