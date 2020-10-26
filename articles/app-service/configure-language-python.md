---
title: 配置 Linux Python 应用
description: 了解如何使用 Azure 门户和 Azure CLI 配置运行 Web 应用的 Python 容器。
ms.topic: quickstart
origin.date: 10/06/2020
ms.date: 10/19/2020
ms.author: v-tawe
ms.reviewer: astay; kraigb
ms.custom: mvc, seodec18, devx-track-python
ms.openlocfilehash: 1986e9170d189a9dd7b6e379b563436a113d585d
ms.sourcegitcommit: e2e418a13c3139d09a6b18eca6ece3247e13a653
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2020
ms.locfileid: "92170828"
---
# <a name="configure-a-linux-python-app-for-azure-app-service"></a>为 Azure 应用服务配置 Linux Python 应用

本文介绍 [Azure 应用服务](overview.md)如何运行 Python 应用，以及如何按需自定义应用服务的行为。 必须连同所有必需的 [pip](https://pypi.org/project/pip/) 模块一起部署 Python 应用。

部署 [Git 存储库](deploy-local-git.md)或 [zip 包](deploy-zip.md)时，应用服务部署引擎会自动激活虚拟环境并运行 `pip install -r requirements.txt`。

对于在应用服务中使用内置 Linux 容器的 Python 开发人员，本指南为其提供了关键概念和说明。 若从未使用过 Azure 应用服务，请先按照 [Python 快速入门](quickstart-python.md)以及[将 Python 与 PostgreSQL 配合使用教程](tutorial-python-postgresql-app.md)进行操作。

可使用 [Azure 门户](https://portal.azure.cn) 或 Azure CLI 进行配置：

- **Azure 门户**：按照[在 Azure 门户配置应用服务应用](configure-common.md)中所述，使用应用的“设置” > 配置”页 。

- **Azure CLI**： 

    - 通过安装最新版的 [Azure CLI](/cli/install-azure-cli) 在本地运行命令，然后使用 [az login](/cli/reference-index#az-login) 登录到 Azure。
    
> [!NOTE]
> Linux 是目前在应用服务中用于运行 Python 应用的推荐选项。 有关 Windows 选项的信息，请参阅 [Windows 风格的应用服务上的 Python](https://docs.microsoft.com/visualstudio/python/managing-python-on-azure-app-service)。

## <a name="configure-python-version"></a>配置 Python 版本

- **Azure 门户**：按照针对 Linux 容器的[配置常规设置](configure-common.md#configure-general-settings)中所述，使用“配置”页上的“常规设置”选项卡 。

- **Azure CLI**：

    -  使用 [az webapp config show](/cli/webapp/config#az_webapp_config_show) 显示当前 Python 版本：
    
        ```azurecli
        az webapp config show --resource-group <resource-group-name> --name <app-name> --query linuxFxVersion
        ```
        
        将 `<resource-group-name>` 和 `<app-name>` 替换为适合 Web 应用的名称。
    
    - 使用 [az webapp config set](/cli/webapp/config#az_webapp_config_set) 设置 Python 版本
        
        ```azurecli
        az webapp config set --resource-group <resource-group-name> --name <app-name> --linux-fx-version "PYTHON|3.7"
        ```
    
    - 使用 [az webapp list-runtimes](/cli/webapp#az_webapp_list_runtimes) 显示 Azure 应用服务中支持的所有 Python 版本：
    
        ```azurecli
        az webapp list-runtimes --linux | grep PYTHON
        ```
    
生成自己的容器映像可以运行不受支持的 Python 版本。

<!-- <a> element here to preserve external links-->
<a name="access-environment-variables"></a>

## <a name="customize-build-automation"></a>自定义生成自动化

在使用 Git 或 zip 包部署应用时，应用服务的生成系统（称为 Oryx）执行以下步骤：

1. 如果由 `PRE_BUILD_COMMAND` 设置指定，请运行预先生成的自定义脚本。
1. 运行 `pip install -r requirements.txt`。 requirements.txt 文件必须存在于项目的根文件夹中。 否则，生成过程将报告错误：“找不到 setup.py 或 requirements.txt；未运行 pip install。”
1. 如果在存储库的根文件夹中找到了 manage.py（指示 Django 应用），则运行 manage.py collectstatic 。 但如果 `DISABLE_COLLECTSTATIC` 设置为 `true`，则跳过此步骤。
1. 如果由 `POST_BUILD_COMMAND` 设置指定，请运行后期生成的自定义脚本。

默认情况下，`PRE_BUILD_COMMAND`、`POST_BUILD_COMMAND` 和 `DISABLE_COLLECTSTATIC` 设置为空。 

- 若要在生成 Django 应用时禁用正在运行的 collectstatic，请将 `DISABLE_COLLECTSTATIC` 设置设置为 true。

- 若要运行预先生成的命令，请将 `PRE_BUILD_COMMAND` 设置设置为包含命令（如 `echo Pre-build command`），或包含与项目根文件夹相对的脚本文件的路径（如 `scripts/prebuild.sh`）。 所有命令都必须使用项目根文件夹的相对路径。

- 若要运行后期生成的命令，请将 `POST_BUILD_COMMAND` 设置设置为包含命令（如 `echo Post-build command`），或包含与项目根文件夹相对的脚本文件的路径（如 `scripts/postbuild.sh`）。 所有命令都必须使用项目根文件夹的相对路径。

有关自定义生成自动化的其他设置，请参阅 [Oryx 配置](https://github.com/microsoft/Oryx/blob/master/doc/configuration.md)。 

若要详细了解应用服务如何在 Linux 中运行和生成 Python 应用，请参阅 [Oryx 如何检测和生成 Python 应用](https://github.com/microsoft/Oryx/blob/master/doc/runtimes/python.md)。

> [!NOTE]
> `PRE_BUILD_SCRIPT_PATH` 和 `POST_BUILD_SCRIPT_PATH` 设置与 `PRE_BUILD_COMMAND` 和 `POST_BUILD_COMMAND` 相同，并且支持用于旧用途。
> 
> 如果名为 `SCM_DO_BUILD_DURING_DEPLOYMENT` 的设置包含 `true` 或 1，该设置会在部署过程中触发 Oryx 生成。 当使用 git、Azure CLI 命令 `az webapp up` 和 Visual Studio Code 进行部署时，此设置为 true。

> [!NOTE]
> 始终在所有预先生成和后期生成脚本中使用相对路径，因为运行 Oryx 的生成容器与运行应用的运行时容器不同。 决不要依赖于应用项目文件夹在容器中的确切位置（例如，其位于 site/wwwroot 下）。

## <a name="production-settings-for-django-apps"></a>Django 应用的生产设置

对于 Azure 应用服务之类的生产环境，Django 应用应遵循 Django 的[部署清单](https://docs.djangoproject.com/en/3.1/howto/deployment/checklist/) (djangoproject.com)。

下表描述了与 Azure 相关的生产设置。 这些设置在应用的 setting.py 文件中定义。

| Django 设置 | Azure 说明 |
| --- | --- |
| `SECRET_KEY` | 如[访问作为环境变量的应用设置](#access-app-settings-as-environment-variables)所述，请将值存储在应用服务设置中。 也可以[在 Azure Key Vault 中将该值存储为“机密”](/key-vault/secrets/quick-create-python)。 |
| `DEBUG` | 在应用服务上创建一个值为 0 (false) 的 `DEBUG` 设置，然后将该值加载为环境变量。 在开发环境中，创建一个值为 1 (true) 的 `DEBUG` 环境变量。 |
| `ALLOWED_HOSTS` | 在生产环境中，Django 要求在 settings.py 的 `ALLOWED_HOSTS` 数组中包含应用的 URL。 可使用 `os.environ['WEBSITE_HOSTNAME']` 代码在运行时检索此 URL。 应用服务会自动将 `WEBSITE_HOSTNAME` 环境变量设置为应用的 URL。 |
| `DATABASES` | 在应用服务中为数据库连接定义设置，并将这些设置加载为环境变量以填充 [`DATABASES`](https://docs.djangoproject.com/en/3.1/ref/settings/#std:setting-DATABASES) 字典。 也可以将值（尤其是用户名和密码）存储为 [Azure Key Vault 机密](/key-vault/secrets/quick-create-python)。 |

## <a name="container-characteristics"></a>容器特征

部署到应用服务时，Python 应用在[应用服务 Python GitHub 存储库](https://github.com/Azure-App-Service/python)中定义的 Linux Docker 容器内运行。 可以在特定于版本的目录中找到映像配置。

此容器具有以下特征：

- 应用是结合附加参数 `--bind=0.0.0.0 --timeout 600`，使用 [Gunicorn WSGI HTTP Server](https://gunicorn.org/) 运行的。
    - 如 [Gunicorn 配置概述](https://docs.gunicorn.org/en/stable/configure.html#configuration-file) (docs.gunicorn.org) 中所述，可通过项目根文件夹中的 gunicorn.conf.py 文件为 Gunicorn 提供配置设置。 也可以[自定义启动命令](#customize-startup-command)。

    - 若要保护 Web 应用免遭意外或蓄意的 DDOS 攻击，请按照[部署 Gunicorn](https://docs.gunicorn.org/en/latest/deploy.html) (docs.gunicorn.org) 中所述，Gunicorn 在 Nginx 反向代理之后运行。

- 默认情况下，基础容器映像仅包含 Flask Web 框架，但容器支持符合 WSGI 规范并与 Python 3.6+ 兼容的其他框架，例如 Django。

- 若要安装其他包（例如 Django），请在项目的根文件夹中创建 [requirements.txt](https://pip.pypa.io/en/stable/user_guide/#requirements-files) 文件，该文件指定直接依赖项。 然后，部署项目时，应用服务会自动安装这些依赖项。

    requirements.txt 文件必须位于项目根文件夹中，依赖项才能安装 。 否则，生成过程将报告错误：“找不到 setup.py 或 requirements.txt；未运行 pip install。” 如果遇到此错误，请检查 requirements 文件的位置。

- 应用服务使用 Web 应用的 URL 自动定义名为 `WEBSITE_HOSTNAME` 的环境变量，例如 `msdocs-hello-world.chinacloudsites.cn`。 它还使用应用的名称定义 `WEBSITE_SITE_NAME`，例如 `msdocs-hello-world`。 
   
## <a name="container-startup-process"></a>容器启动过程

在启动期间，Linux 上的应用服务容器将运行以下步骤：

1. 使用[自定义启动命令](#customize-startup-command)（如果已提供）。
2. 检查是否存在 [Django 应用](#django-app)，如果已检测到，请为其启动 Gunicorn。
3. 检查是否存在 [Flask 应用](#flask-app)，如果已检测到，请为其启动 Gunicorn。
4. 如果未找到其他任何应用，则启动容器中内置的默认应用。

以下部分提供了每个选项的更多详细信息。

### <a name="django-app"></a>Django 应用

对于 Django 应用，应用服务将在应用代码中查找名为 `wsgi.py` 的文件，然后使用以下命令运行 Gunicorn：

```bash
# <module> is the name of the folder that contains wsgi.py
gunicorn --bind=0.0.0.0 --timeout 600 <module>.wsgi
```

若要更精细地控制启动命令，请使用[自定义启动命令](#customize-startup-command)，将 `<module>` 替换为包含 wsgi.py 的文件夹名称，如果该模块不在项目根文件夹中，则添加 `--chdir` 参数。 例如，如果 wsgi.py 位于项目根文件夹中的 knboard/backend/config 下，请使用参数 `--chdir knboard/backend config.wsgi` 。

若要启用生产日志记录，请按照[自定义启动命令](#customize-startup-command)示例中所示，添加 `--access-logfile` 和 `--error-logfile` 参数。

### <a name="flask-app"></a>Flask 应用

对于 Flask，应用服务将查找名为 *application.py* 或 *app.py* 的文件并启动 Gunicorn，如下所示：

```bash
# If application.py
gunicorn --bind=0.0.0.0 --timeout 600 application:app

# If app.py
gunicorn --bind=0.0.0.0 --timeout 600 app:app
```

如果主应用模块包含在不同的文件中，请对应用对象使用不同的名称；若要为 Gunicorn 提供附加的参数，请使用[自定义启动命令](#customize-startup-command)。

### <a name="default-behavior"></a>默认行为

如果应用服务找不到自定义命令、Django 应用或 Flask 应用，则它会运行位于 _opt/defaultsite_ 文件夹中的默认只读应用。 默认应用如下所示：

![Linux 上的默认应用服务网页](media/configure-language-python/default-python-app.png)

## <a name="customize-startup-command"></a>自定义启动命令

如本文前面所述，可以通过项目根文件夹中的 gunicorn.conf.py 文件为 Gunicorn 提供配置设置，如 [Gunicorn 配置概述](https://docs.gunicorn.org/en/stable/configure.html#configuration-file)中所述。

如果此类配置不够，可以通过在启动命令文件中提供自定义启动命令或多个命令来控制容器的启动行为。 启动命令文件可使用你选择的任何名称，例如 startup.sh、startup.cmd、startup.txt 等  。

所有命令都必须使用项目根文件夹的相对路径。

指定启动命令或命令文件：

- **Azure 门户**：选择应用的“配置”页，然后选择“常规设置” 。 在“启动命令”字段中，输入启动命令的全文或启动命令文件的名称。 然后，选择“保存”，应用所做的更改。 请参阅针对 Linux 容器的[配置常规设置](configure-common.md#configure-general-settings)。

- **Azure CLI**：将 [az webapp config set](/cli/webapp/config#az_webapp_config_set) 命令与 `--startup-file` 参数一起使用，以设置启动命令或文件：

```azurecli
    az webapp config set --resource-group <resource-group-name> --name <app-name> --startup-file "<custom-command>"
    ```
        
    Replace `<custom-command>` with either the full text of your startup command or the name of your startup command file.
        
App Service ignores any errors that occur when processing a custom startup command or file, then continues its startup process by looking for Django and Flask apps. If you don't see the behavior you expect, check that your startup command or file is error-free and that a startup command file is deployed to App Service along with your app code. You can also check the [Diagnostic logs](#access-diagnostic-logs) for additional information. Also check the app's **Diagnose and solve problems** page on the [Azure portal](https://portal.azure.cn).

### Example startup commands

- **Added Gunicorn arguments**: The following example adds the `--workers=4` to a Gunicorn command line for starting a Django app: 

    ```bash
    # <module-path> is the relative path to the folder that contains the module
    # that contains wsgi.py; <module> is the name of the folder containing wsgi.py.
    gunicorn --bind=0.0.0.0 --timeout 600 --workers=4 --chdir <module_path> <module>.wsgi
    ```    

    For more information, see [Running Gunicorn](https://docs.gunicorn.org/en/stable/run.html) (docs.gunicorn.org).

- **Enable production logging for Django**: Add the `--access-logfile '-'` and `--error-logfile '-'` arguments to the command line:

    ```bash    
    # '-' for the log files means stdout for --access-logfile and stderr for --error-logfile.
    gunicorn --bind=0.0.0.0 --timeout 600 --workers=4 --chdir <module_path> <module>.wsgi --access-logfile '-' --error-logfile '-'
    ```    

    These logs will appear in the [App Service log stream](#access-diagnostic-logs).

    For more information, see [Gunicorn logging](https://docs.gunicorn.org/en/stable/settings.html#logging) (docs.gunicorn.org).
    
- **Custom Flask main module**: by default, App Service assumes that a Flask app's main module is *application.py* or *app.py*. If your main module uses a different name, then you must customize the startup command. For example, yf you have a Flask app whose main module is *hello.py* and the Flask app object in that file is named `myapp`, then the command is as follows:

    ```bash
    gunicorn --bind=0.0.0.0 --timeout 600 hello:myapp
    ```
    
    If your main module is in a subfolder, such as `website`, specify that folder with the `--chdir` argument:
    
    ```bash
    gunicorn --bind=0.0.0.0 --timeout 600 --chdir website hello:myapp
    ```
    
- **Use a non-Gunicorn server**: To use a different web server, such as [aiohttp](https://aiohttp.readthedocs.io/en/stable/web_quickstart.html), use the appropriate command as the startup command or in the startup command file:

    ```bash
    python3.7 -m aiohttp.web -H localhost -P 8080 package.module:init_func
    ```

## Access app settings as environment variables

App settings are values stored in the cloud specifically for your app as described on [Configure app settings](configure-common.md#configure-app-settings). These settings are available to your app code as environment variables and accessed using the standard [os.environ](https://docs.python.org/3/library/os.html#os.environ) pattern.

For example, if you've created app setting called `DATABASE_SERVER`, the following code retrieves that setting's value:

```python
db_server = os.environ['DATABASE_SERVER']
```
    
## <a name="detect-https-session"></a>检测 HTTPS 会话

在应用服务中，SSL 终止在网络负载均衡器上发生，因此，所有 HTTPS 请求将以未加密的 HTTP 请求形式访问你的应用。 如果应用逻辑需要检查用户请求是否已加密，可以检查 `X-Forwarded-Proto` 标头。

```python
if 'X-Forwarded-Proto' in request.headers and request.headers['X-Forwarded-Proto'] == 'https':
# Do something when HTTPS is used
```

使用常用 Web 框架可以访问采用标准应用模式的 `X-Forwarded-*` 信息。 在 [CodeIgniter](https://codeigniter.com/) 中，[is_https()](https://github.com/bcit-ci/CodeIgniter/blob/master/system/core/Common.php#L338-L365) 默认检查 `X_FORWARDED_PROTO` 的值。

## <a name="access-diagnostic-logs"></a>访问诊断日志

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

若要通过 Azure 门户访问日志，请在应用的左侧菜单中选择“监视” > “日志流” 。

## <a name="open-ssh-session-in-browser"></a>在浏览器中打开 SSH 会话

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-builtin-no-h.md)]

## <a name="troubleshooting"></a>疑难解答

- **部署自己的应用代码后看到默认应用。** 之所以显示默认应用，是因为并未将应用代码部署到应用服务，或者应用服务未找到应用代码，因此运行了默认应用。

    - 请重启应用服务，等待 15 到 20 秒，然后再次检查应用。
    
    - 请确保使用适用于 Linux 的应用服务，而不要使用基于 Windows 的实例。 在 Azure CLI 中运行 `az webapp show --resource-group <resource-group-name> --name <app-name> --query kind` 命令，对 `<resource-group-name>` 和 `<app-service-name>` 进行相应的替换。 应该会看到作为输出的 `app,linux`，否则请重新创建应用服务并选择 Linux。
    
    - 使用 SSH 或 Kudu 控制台直接连接到应用服务，并检查文件是否存在于 *site/wwwroot* 下。 如果这些文件不存在，请检查部署过程并重新部署应用。
    
    - 如果这些文件存在，则表示应用服务无法识别特定的启动文件。 检查是否按应用服务的预期方式为 [Django](#django-app) 或 [Flask](#flask-app) 构建了应用，或使用[自定义启动命令](#customize-startup-command)。

- **浏览器中显示“服务不可用”消息。** 浏览器在等待应用服务的响应时超时，这表示应用服务已启动 Gunicorn 服务器，但指定应用代码的参数不正确。

    - 刷新浏览器，尤其是在应用服务计划中使用最低定价层的情况下。 例如，使用免费层时，应用可能需要较长时间才能启动，并在刷新浏览器后才会做出响应。

    - 检查是否按应用服务的预期方式为 [Django](#django-app) 或 [Flask](#flask-app) 构建了应用，或使用[自定义启动命令](#customize-startup-command)。

    - 检查“[日志流](#access-diagnostic-logs)”是否有任何错误消息。

- 日志流显示“找不到 setup.py 或 requirements.txt；未运行 pip install。”：Oryx 生成过程找不到 requirements.txt 文件。

    - 使用 SSH 或 Kudu 控制台直接连接到应用服务，并验证 requirements.txt 是否直接存在于 site/wwwroot 下 。 如果该文件不存在，请使该文件存在于存储库中，并包含在部署中。 如果它存在于单独的文件夹中，请将其移到根文件夹下。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [教程：使用 PostgreSQL 的 Python 应用](tutorial-python-postgresql-app.md)

<!-- > [!div class="nextstepaction"]
> [Tutorial: Deploy from private container repository](tutorial-custom-container.md?pivots=container-linux) -->

> [!div class="nextstepaction"]
> [应用服务 Linux 常见问题解答](faq-app-service-linux.md)
