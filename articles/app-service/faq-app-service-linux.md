---
title: 运行内置容器常见问题解答
description: 查找有关 Azure 应用服务中内置 Linux 容器的常见问题的解答。
keywords: Azure 应用服务, Web 应用, 常见问题解答, Linux, oss, 用于容器的 Web 应用, 多容器, 多容器
author: msangapu-msft
ms.topic: article
origin.date: 10/30/2018
ms.date: 10/19/2020
ms.author: v-tawe
ms.custom: seodec18
ms.openlocfilehash: 6a6d60e420585f546d04c29856c65244708433b2
ms.sourcegitcommit: e2e418a13c3139d09a6b18eca6ece3247e13a653
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2020
ms.locfileid: "92170687"
---
# <a name="azure-app-service-on-linux-faq"></a>Linux 上的 Azure 应用服务常见问题解答

随着 Linux 应用服务的发布，我们正在努力添加功能和改进我们的平台。 本文提供客户最近提出的问题的解答。

如果你有问题，请对本文发表评论。

<!-- ## Built-in images

**I want to fork the built-in Docker containers that the platform provides. Where can I find those files?**

You can find all Docker files on [GitHub](https://github.com/azure-app-service). You can find all Docker containers on [Docker Hub](https://hub.docker.com/u/appsvc/).

<a id="#startup-file"></a>

**What are the expected values for the Startup File section when I configure the runtime stack?**

| Stack           | Expected Value                                                                         |
|-----------------|----------------------------------------------------------------------------------------|
| Java SE         | the command to start your JAR app (for example, `java -jar /home/site/wwwroot/app.jar --server.port=80`) |
| Tomcat          | the location of a script to perform any necessary configurations (for example, `/home/site/deployments/tools/startup_script.sh`)          |
| Node.js         | the PM2 configuration file or your script file                                |
| .NET Core       | the compiled DLL name as `dotnet <myapp>.dll`                                 |
| Ruby            | the Ruby script that you want to initialize your app with                     |

These commands or scripts are executed after the built-in Docker container is started, but before your application code is started. -->

## <a name="management"></a>管理

**在 Azure 门户中按下“重启”按钮时，会发生什么情况？**

此操作等同于 Docker 重启。

**可以使用安全外壳 (SSH) 连接到应用容器虚拟机 (VM) 吗？**

是的，可以通过源代码管理 (SCM) 站点实现此操作。

> [!NOTE]
> 还可以使用 SSH、SFTP 或 Visual Studio Code（用于实时调试 Node.js 应用）直接从本地开发计算机连接到应用容器。 有关详细信息，请参阅[远程调试和通过 SSH 登录到 Linux 上的应用服务](https://azure.github.io/AppService/2018/05/07/New-SSH-Experience-and-Remote-Debugging-for-Linux-Web-Apps.html)。
>

**如何通过 SDK 或 Azure 资源管理器模板创建 Linux 应用服务计划？**

应将应用服务的“保留”字段设置为“true”。

## <a name="continuous-integration-and-deployment"></a>持续集成和持续部署

<!-- **My web app still uses an old Docker container image after I've updated the image on Docker Hub. Do you support continuous integration and deployment of custom containers?**

Yes, to set up continuous integration/deployment for Azure Container Registry or DockerHub, by following [Continuous Deployment with Web App for Containers](./deploy-ci-cd-custom-container.md). For private registries, you can refresh the container by stopping and then starting your web app. Or you can change or add a dummy application setting to force a refresh of your container. -->

**是否支持过渡环境？**

是的。

**是否可以使用 *WebDeploy/MSDeploy* 来部署 Web 应用？**

可以，需要将名为 `WEBSITE_WEBDEPLOY_USE_SCM` 的应用设置设置为 *false*。

**使用 Linux Web 应用时，应用程序的 Git 部署失败。如何解决此问题？**

如果 Linux Web 应用的 Git 部署失败，可选择以下选项之一部署应用程序代码：

- 使用持续交付（预览版）功能：可将应用的源代码存储在 Azure DevOps Git 存储库或 GitHub 存储库中，以使用 Azure 持续交付。 有关详细信息，请参阅[如何为 Linux Web 应用配置持续交付](https://blogs.msdn.microsoft.com/devops/2017/05/10/use-azure-portal-to-setup-continuous-delivery-for-web-app-on-linux/)。

- 使用 [ZIP 部署 API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file)：若要使用此 API，请[通过 SSH 连接到 Web 应用](configure-linux-open-ssh-session.md)，并转到要在其中部署代码的文件夹。 运行以下代码：

   ```bash
   curl -X POST -u <user> --data-binary @<zipfile> https://{your-sitename}.scm.chinacloudsites.cn/api/zipdeploy
   ```

   如果有错误指出找不到 `curl` 命令，请确保在运行前一条 `curl` 命令之前使用 `apt-get install curl` 安装 curl。

## <a name="language-support"></a>语言支持

**我想要在 Node.js 应用程序中使用 Web 套接字，要设置什么特殊设置或配置吗？**

需要，请在服务器端 Node.js 代码中禁用 `perMessageDeflate`。 例如，如果要使用 socket.io，请使用以下代码：

```nodejs
const io = require('socket.io')(server,{
  perMessageDeflate :false
});
```

**是否支持未编译的 .NET Core 应用？**

是的。

**是否支持将 Composer 用作 PHP 应用的依赖关系管理器？**

支持，在 Git 部署过程中，Kudu 应检测到正在部署 PHP 应用程序（这得益于 composer.lock 文件的存在），然后触发 composer 安装。

<!-- ## Custom containers

**I'm using my own custom container. I want the platform to mount an SMB share to the `/home/` directory.**

If `WEBSITES_ENABLE_APP_SERVICE_STORAGE` setting is **unspecified** or set to *true*, the `/home/` directory **will be shared** across scale instances, and files written **will persist** across restarts. Explicitly setting `WEBSITES_ENABLE_APP_SERVICE_STORAGE` to *false* will disable the mount.

**My custom container takes a long time to start, and the platform restarts the container before it finishes starting up.**

You can configure the amount of time the platform will wait before it restarts your container. To do so, set the `WEBSITES_CONTAINER_START_TIME_LIMIT` app setting to the value you want. The default value is 230 seconds, and the maximum value is 1800 seconds.

**What is the format for the private registry server URL?**

Provide the full registry URL, including `http://` or `https://`.

**What is the format for the image name in the private registry option?**

Add the full image name, including the private registry URL (for example, myacr.azurecr.cn/dotnet:latest). Image names that use a custom port [cannot be entered through the portal](https://feedback.azure.com/forums/169385-web-apps/suggestions/31304650). To set `docker-custom-image-name`, use the [`az` command-line tool](https://docs.microsoft.com/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set).

**Can I expose more than one port on my custom container image?**

We don't support exposing more than one port.

**Can I bring my own storage?**

Yes, [bring your own storage](https://docs.microsoft.com/azure/app-service/configure-connect-to-azure-storage) is in preview.

**Why can't I browse my custom container's file system or running processes from the SCM site?**

The SCM site runs in a separate container. You can't check the file system or running processes of the app container.

**My custom container listens to a port other than port 80. How can I configure my app to route requests to that port?**

We have automatic port detection. You can also specify an app setting called *WEBSITES_PORT* and give it the value of the expected port number. Previously, the platform used the *PORT* app setting. We are planning to deprecate this app setting and to use *WEBSITES_PORT* exclusively.

**Do I need to implement HTTPS in my custom container?**

No, the platform handles HTTPS termination at the shared front ends. -->

<!-- ## Multi-container with Docker Compose

**How do I configure Azure Container Registry (ACR) to use with multi-container?**

In order to use ACR with multi-container, **all container images** need to be hosted on the same ACR registry server. Once they are on the same registry server, you will need to create application settings and then update the Docker Compose configuration file to include the ACR image name.

Create the following application settings:

- DOCKER_REGISTRY_SERVER_USERNAME
- DOCKER_REGISTRY_SERVER_URL (full URL, ex: `https://<server-name>.azurecr.cn`)
- DOCKER_REGISTRY_SERVER_PASSWORD (enable admin access in ACR settings)

Within the configuration file, reference your ACR image like the following example:

```yaml
image: <server-name>.azurecr.cn/<image-name>:<tag>
```

**How do I know which container is internet accessible?**

- Only one container can be open for access
- Only port 80 and 8080 is accessible (exposed ports)

Here are the rules for determining which container is accessible - in the order of precedence:

- Application setting `WEBSITES_WEB_CONTAINER_NAME` set to the container name
- The first container to define port 80 or 8080
- If neither of the above is true, the first container defined in the file will be accessible (exposed) -->


## <a name="web-sockets"></a>Web 套接字

Linux 应用支持 Web 套接字。

> [!IMPORTANT]
> 免费应用服务计划中的 Linux 应用目前不支持 Web 套接字。 我们正在努力消除此限制，并计划在免费应用计划中最多支持 5 个 Web 套接字连接。

## <a name="pricing-and-sla"></a>定价和 SLA

**服务正式版的定价是多少？**

定价因 SKU 和区域而异，但你可以在我们的定价页上查看更多详细信息：[应用服务定价](https://www.azure.cn/pricing/details/app-service/)。

## <a name="other-questions"></a>其他问题

**“请求的功能在资源组中不可用”是什么意思？**

使用 Azure 资源管理器 (ARM) 创建 Web 应用时，你可能会看到此消息。 根据当前的限制，对于同一资源组，不能在同一区域中混合使用 Windows 和 Linux 应用。

**应用程序设置名称中支持的字符有哪些？**

应用程序设置只能使用字母（A-Z、a-z）、数字 (0-9) 和下划线字符 (_)。

**可在何处请求新功能？**

可以在 [Web 应用反馈论坛](https://aka.ms/webapps-uservoice)提交看法。 请将“[Linux]”添加到建议的标题中。

## <a name="next-steps"></a>后续步骤

- [什么是 Linux 上的 Azure 应用服务？](overview.md#app-service-on-linux)
- [设置 Azure 应用服务中的过渡环境](deploy-staging-slots.md)

<!-- - [Continuous Deployment with Web App for Containers](./deploy-ci-cd-custom-container.md) -->
