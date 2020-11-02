---
title: 使用 GitHub Actions 在 Azure Functions 中进行代码更新
description: 了解如何使用 GitHub Actions 来定义一个在 GitHub 中生成和部署 Azure Functions 项目的工作流。
author: craigshoemaker
ms.topic: conceptual
ms.date: 10/19/2020
ms.author: v-junlch
ms.custom: devx-track-csharp, github-actions-azure
ms.openlocfilehash: 8cf307d503d3a95d9c373f458ba6b0b927581a14
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472578"
---
# <a name="continuous-delivery-by-using-github-action"></a>使用 Github Actions 进行持续交付

使用 [GitHub Actions](https://github.com/features/actions) 定义一个工作流，以便自动生成代码并将其部署到 Azure 函数应用。 

在 GitHub Actions 中，[工作流](https://help.github.com/articles/about-github-actions#workflow)是在 GitHub 存储库中定义的自动化过程。 此过程告知 GitHub 如何在 GitHub 中生成和部署函数应用项目。 

工作流通过存储库的 `/.github/workflows/` 路径中的 YAML (.yml) 文件定义。 此定义包含组成工作流的各种步骤和参数。 

对于 Azure Functions 工作流，此文件有三个部分： 

| 部分 | 任务 |
| ------- | ----- |
| **身份验证** | <ol><li>下载发布配置文件或定义服务主体。</li><li>创建 GitHub 机密。</li></ol>|
| **生成** | <ol><li>设置环境。</li><li>生成函数应用。</li></ol> |
| **部署** | <ol><li>部署函数应用。</li></ol>|

> [!NOTE]
> 如果决定使用发布配置文件进行身份验证，则不需创建服务主体。

## <a name="downloading-and-using-a-publish-profile-as-deployment-credential-recommended"></a>下载发布配置文件并将其用作部署凭据（推荐）

若要下载函数应用的发布配置文件：

1. 选择函数应用的“概述”页，然后选择“获取发布配置文件”   。

   :::image type="content" source="media/functions-how-to-github-actions/get-publish-profile.png" alt-text="下载发布配置文件":::

1. 保存并复制发布设置文件的内容。

## <a name="create-a-service-principal-deprecated"></a>创建服务主体（已弃用）

## <a name="prerequisites"></a>先决条件

- 具有活动订阅的 Azure 帐户。 [创建帐户](https://www.azure.cn/pricing/1rmb-trial)
- 一个 GitHub 帐户。 如果没有该帐户，请注册[免费版](https://github.com/join)。  
- 在 Azure 上托管的使用 GitHub 存储库的工作函数应用。   
    - [快速入门：在 Azure 中使用 Visual Studio Code 创建函数](functions-create-first-function-vs-code.md)


## <a name="generate-deployment-credentials"></a>生成部署凭据

对于 GitHub Actions，若要通过 Azure Functions 进行身份验证，建议使用发布配置文件。 也可以使用服务主体进行身份验证，但该过程需要更多步骤。 

## <a name="configure-the-github-secret"></a>配置 GitHub 机密

将发布配置文件凭据或服务主体另存为 [GitHub 机密](https://docs.github.com/en/actions/reference/encrypted-secrets)，以便使用 Azure 进行身份验证。 你将在工作流中访问机密。 

# <a name="publish-profile"></a>[发布配置文件](#tab/publish-profile)

若要下载函数应用的发布配置文件：

1. 选择函数应用的“概述”页，然后选择“获取发布配置文件”   。

   :::image type="content" source="media/functions-how-to-github-actions/get-publish-profile.png" alt-text="下载发布配置文件":::

1. 保存并复制该文件的内容。


# <a name="service-principal"></a>[服务主体](#tab/service-principal)

可以使用 [Azure CLI](/cli/) 的 [az ad sp create-for-rbac](/cli/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac&preserve-view=true) 命令创建[服务主体](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object)。 

```azurecli
az ad sp create-for-rbac --name "<MY-APP-NAME>" --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.Web/sites/<APP_NAME> --sdk-auth
```

在此示例中，请将资源中的占位符替换为你的订阅 ID、资源组以及函数应用名称。 输出是用于访问函数应用的角色分配凭据。 请复制此 JSON 对象，它可以用来从 GitHub 进行身份验证。 

```output 
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

> [!IMPORTANT]
> 始终应授予最小访问权限。 这是在上一示例中将范围限制为特定的函数应用而不是整个资源组的原因。

---

## <a name="add-the-github-secret"></a>添加 GitHub 机密

1. 在  >  “机密” >  “添加新机密”。

   :::image type="content" source="media/functions-how-to-github-actions/add-secret.png" alt-text="下载发布配置文件":::

1. 添加新机密。

   * 如果使用的是通过使用 Azure CLI 创建的服务主体，请使用 `AZURE_CREDENTIALS` 作为“名称”  。 然后，将复制的 JSON 对象输出粘贴为“值”，并选择“添加机密”   。
   * 如果使用的是发布配置文件，请使用 `AZURE_FUNCTIONAPP_PUBLISH_PROFILE` 作为“名称”  。 然后使用发布配置文件的文件内容作为“值”，并选择“添加机密”   。

GitHub 现在可以针对 Azure 中的函数应用进行身份验证了。

## <a name="create-the-environment"></a>创建环境 

设置环境是使用特定于语言的发布设置操作完成的。

|**语言**  |**设置操作**  |
|---------|---------|
|**.NET**     | `actions/setup-dotnet` |
|**ASP.NET**     | `actions/setup-dotnet` |
|**Java**     | `actions/setup-java` |
|**JavaScript** | `actions/setup-node` |


# <a name="net"></a>[.NET](#tab/dotnet)

以下示例显示了使用 `actions/setup-dotnet` 操作设置环境的工作流部分：

```yaml
    - name: Setup DotNet 2.2.402 Environment
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 2.2.402
```

# <a name="java"></a>[Java](#tab/java)

以下示例显示了使用 `actions/setup-java` 操作设置环境的工作流部分：

```yaml
    - name: Setup Java 1.8.x
      uses: actions/setup-java@v1
      with:
        # If your pom.xml <maven.compiler.source> version is not in 1.8.x
        # Please change the Java version to match the version in pom.xml <maven.compiler.source>
        java-version: '1.8.x'
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

以下示例显示了使用 `actions/setup-node` 操作设置环境的工作流部分：

```yaml

    - name: Setup Node 12.x Environment
      uses: actions/setup-node@v1
      with:
        node-version: 12.x
```

---

## <a name="build-the-function-app"></a>生成函数应用

这取决于语言。对于 Azure Functions 支持的语言，应该可以将此部分视为每种语言的标准生成步骤。

以下示例显示了生成函数应用的工作流部分，该部分是特定于语言的：

# <a name="net"></a>[.NET](#tab/dotnet)

```yaml
    env:
      AZURE_FUNCTIONAPP_PACKAGE_PATH: '.' # set this to the path to your web app project, defaults to the repository root

    - name: 'Resolve Project Dependencies Using Dotnet'
      shell: bash
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        dotnet build --configuration Release --output ./output
        popd
```

# <a name="java"></a>[Java](#tab/java)

```yaml
    env:
      POM_XML_DIRECTORY: '.'  # set this to the directory which contains pom.xml file

    - name: 'Restore Project Dependencies Using Mvn'
      shell: bash
      run: |
        pushd './${{ env.POM_XML_DIRECTORY }}'
        mvn clean package
        mvn azure-functions:package
        popd
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```yaml
    env:
      AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'  # set this to the path to your web app project, defaults to the repository root

    - name: 'Resolve Project Dependencies Using Npm'
      shell: bash
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        npm install
        npm run build --if-present
        npm run test --if-present
        popd
```

---

## <a name="deploy-the-function-app"></a>部署函数应用
使用 `Azure/functions-action` 操作将代码部署到函数应用。 此操作有三个参数：

|参数 |说明  |
|---------|---------|
|_**app-name**_ | （必需）函数应用的名称。 |
|_**slot-name**_ | （可选）要部署到其中的[部署槽](functions-deployment-slots.md)的名称。 该槽必须已经在函数应用中定义。 |
|_**publish-profile**_ | （可选）发布配置文件的 GitHub 机密的名称。 |


### <a name="publish-profile-deploy"></a>发布配置文件部署

以下示例使用 `functions-action` 版本 1 和 `publish profile` 进行身份验证：


# <a name="net"></a>[.NET](#tab/dotnet)

设置使用发布配置文件的 .NET Linux 工作流。

```yaml
name: Deploy DotNet project to Azure function app with a Linux environment

on:
  [push]

env:
  AZURE_FUNCTIONAPP_NAME: your-app-name  # set this to your application's name
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'    # set this to the path to your web app project, defaults to the repository root
  DOTNET_VERSION: '2.2.402'              # set this to the dotnet version to use

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: Setup DotNet ${{ env.DOTNET_VERSION }} Environment
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    - name: 'Resolve Project Dependencies Using Dotnet'
      shell: bash
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        dotnet build --configuration Release --output ./output
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/output'
        publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}
```
设置使用发布配置文件的 .NET Windows 工作流。

```yaml
name: Deploy DotNet project to Azure function app with a Windows environment

on:
  [push]

env:
  AZURE_FUNCTIONAPP_NAME: your-app-name  # set this to your application's name
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'    # set this to the path to your web app project, defaults to the repository root
  DOTNET_VERSION: '2.2.402'              # set this to the dotnet version to use

jobs:
  build-and-deploy:
    runs-on: windows-latest
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: Setup DotNet ${{ env.DOTNET_VERSION }} Environment
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    - name: 'Resolve Project Dependencies Using Dotnet'
      shell: pwsh
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        dotnet build --configuration Release --output ./output
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/output'
        publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}
```

# <a name="java"></a>[Java](#tab/java)

设置使用发布配置文件的 Java Linux 工作流。

```yaml
name: Deploy Java project to Azure Function App

on:
  [push]

env:
  AZURE_FUNCTIONAPP_NAME: your-app-name      # set this to your function app name on Azure
  POM_XML_DIRECTORY: '.'                     # set this to the directory which contains pom.xml file
  POM_FUNCTIONAPP_NAME: your-app-name        # set this to the function app name in your local development environment
  JAVA_VERSION: '1.8.x'                      # set this to the dotnet version to use

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: Setup Java Sdk ${{ env.JAVA_VERSION }}
      uses: actions/setup-java@v1
      with:
        java-version: ${{ env.JAVA_VERSION }}

    - name: 'Restore Project Dependencies Using Mvn'
      shell: bash
      run: |
        pushd './${{ env.POM_XML_DIRECTORY }}'
        mvn clean package
        mvn azure-functions:package
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: './${{ env.POM_XML_DIRECTORY }}/target/azure-functions/${{ env.POM_FUNCTIONAPP_NAME }}'
        publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}
```

设置使用发布配置文件的 Java Windows 工作流。

```yaml
name: Deploy Java project to Azure Function App

on:
  [push]

env:
  AZURE_FUNCTIONAPP_NAME: your-app-name      # set this to your function app name on Azure
  POM_XML_DIRECTORY: '.'                     # set this to the directory which contains pom.xml file
  POM_FUNCTIONAPP_NAME: your-app-name        # set this to the function app name in your local development environment
  JAVA_VERSION: '1.8.x'                      # set this to the java version to use

jobs:
  build-and-deploy:
    runs-on: windows-latest
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: Setup Java Sdk ${{ env.JAVA_VERSION }}
      uses: actions/setup-java@v1
      with:
        java-version: ${{ env.JAVA_VERSION }}

    - name: 'Restore Project Dependencies Using Mvn'
      shell: pwsh
      run: |
        pushd './${{ env.POM_XML_DIRECTORY }}'
        mvn clean package
        mvn azure-functions:package
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: './${{ env.POM_XML_DIRECTORY }}/target/azure-functions/${{ env.POM_FUNCTIONAPP_NAME }}'
        publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

设置使用发布配置文件的 Node.JS Linux 工作流。

```yaml
name: Deploy Node.js project to Azure Function App

on:
  [push]

env:
  AZURE_FUNCTIONAPP_NAME: your-app-name    # set this to your application's name
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '12.x'                     # set this to the node version to use (supports 8.x, 10.x, 12.x)

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: Setup Node ${{ env.NODE_VERSION }} Environment
      uses: actions/setup-node@v1
      with:
        node-version: ${{ env.NODE_VERSION }}

    - name: 'Resolve Project Dependencies Using Npm'
      shell: bash
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        npm install
        npm run build --if-present
        npm run test --if-present
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: ${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}
        publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}
```

设置使用发布配置文件的 Node.JS Windows 工作流。

```yaml
name: Deploy Node.js project to Azure Function App

on:
  [push]

env:
  AZURE_FUNCTIONAPP_NAME: your-app-name    # set this to your application's name
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '10.x'                     # set this to the node version to use (supports 8.x, 10.x, 12.x)

jobs:
  build-and-deploy:
    runs-on: windows-latest
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: Setup Node ${{ env.NODE_VERSION }} Environment
      uses: actions/setup-node@v1
      with:
        node-version: ${{ env.NODE_VERSION }}

    - name: 'Resolve Project Dependencies Using Npm'
      shell: pwsh
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        npm install
        npm run build --if-present
        npm run test --if-present
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: ${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}
        publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}

```
---

### <a name="service-principal-deploy"></a>服务主体部署

以下示例使用 `functions-action` 版本 1 和 `service principal` 进行身份验证。 此工作流会设置 Windows .NET 环境。 

# <a name="net"></a>[.NET](#tab/dotnet)

设置使用服务主体的 .NET Linux 工作流。

```yaml
name: Deploy DotNet project to Azure function app with a Linux environment

on:
  [push]

env:
  AZURE_FUNCTIONAPP_NAME: your-app-name  # set this to your application's name
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'    # set this to the path to your web app project, defaults to the repository root
  DOTNET_VERSION: '2.2.402'              # set this to the dotnet version to use

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: 'Login via Azure CLI'
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Setup DotNet ${{ env.DOTNET_VERSION }} Environment
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    - name: 'Resolve Project Dependencies Using Dotnet'
      shell: bash
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        dotnet build --configuration Release --output ./output
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/output'

     - name: logout
        run: |
          az logout
```

设置使用服务主体的 .NET Windows 工作流。

```yaml
name: Deploy DotNet project to Azure function app with a Windows environment

on:
  [push]

env:
  AZURE_FUNCTIONAPP_NAME: your-app-name  # set this to your application's name
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'    # set this to the path to your web app project, defaults to the repository root
  DOTNET_VERSION: '2.2.402'              # set this to the dotnet version to use

jobs:
  build-and-deploy:
    runs-on: windows-latest
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: 'Login via Azure CLI'
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Setup DotNet ${{ env.DOTNET_VERSION }} Environment
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    - name: 'Resolve Project Dependencies Using Dotnet'
      shell: pwsh
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        dotnet build --configuration Release --output ./output
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/output'

     - name: logout
        run: |
          az logout
```

# <a name="java"></a>[Java](#tab/java)

设置使用服务主体的 Java Linux 工作流。

```yaml
name: Deploy Java project to Azure Function App

on:
  [push]

env:
  AZURE_FUNCTIONAPP_NAME: your-app-name      # set this to your function app name on Azure
  POM_XML_DIRECTORY: '.'                     # set this to the directory which contains pom.xml file
  POM_FUNCTIONAPP_NAME: your-app-name        # set this to the function app name in your local development environment
  JAVA_VERSION: '1.8.x'                      # set this to the dotnet version to use

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: 'Login via Azure CLI'
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}


    - name: Setup Java Sdk ${{ env.JAVA_VERSION }}
      uses: actions/setup-java@v1
      with:
        java-version: ${{ env.JAVA_VERSION }}

    - name: 'Restore Project Dependencies Using Mvn'
      shell: bash
      run: |
        pushd './${{ env.POM_XML_DIRECTORY }}'
        mvn clean package
        mvn azure-functions:package
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: './${{ env.POM_XML_DIRECTORY }}/target/azure-functions/${{ env.POM_FUNCTIONAPP_NAME }}'

     - name: logout
        run: |
          az logout
```

设置使用服务主体的 Java Windows 工作流。

```yaml
name: Deploy Java project to Azure Function App

on:
  [push]

env:
  AZURE_FUNCTIONAPP_NAME: your-app-name      # set this to your function app name on Azure
  POM_XML_DIRECTORY: '.'                     # set this to the directory which contains pom.xml file
  POM_FUNCTIONAPP_NAME: your-app-name        # set this to the function app name in your local development environment
  JAVA_VERSION: '1.8.x'                      # set this to the java version to use

jobs:
  build-and-deploy:
    runs-on: windows-latest
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: 'Login via Azure CLI'
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Setup Java Sdk ${{ env.JAVA_VERSION }}
      uses: actions/setup-java@v1
      with:
        java-version: ${{ env.JAVA_VERSION }}

    - name: 'Restore Project Dependencies Using Mvn'
      shell: pwsh
      run: |
        pushd './${{ env.POM_XML_DIRECTORY }}'
        mvn clean package
        mvn azure-functions:package
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: './${{ env.POM_XML_DIRECTORY }}/target/azure-functions/${{ env.POM_FUNCTIONAPP_NAME }}'

     - name: logout
        run: |
          az logout
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

设置使用服务主体的 Node.JS Linux 工作流。

```yaml
name: Deploy Node.js project to Azure Function App

on:
  [push]

env:
  AZURE_FUNCTIONAPP_NAME: your-app-name    # set this to your application's name
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '12.x'                     # set this to the node version to use (supports 8.x, 10.x, 12.x)

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: 'Login via Azure CLI'
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Setup Node ${{ env.NODE_VERSION }} Environment
      uses: actions/setup-node@v1
      with:
        node-version: ${{ env.NODE_VERSION }}

    - name: 'Resolve Project Dependencies Using Npm'
      shell: bash
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        npm install
        npm run build --if-present
        npm run test --if-present
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: ${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}

     - name: logout
        run: |
          az logout
```

设置使用服务主体的 Node.JS Windows 工作流。

```yaml
name: Deploy Node.js project to Azure Function App

on:
  [push]

env:
  AZURE_FUNCTIONAPP_NAME: your-app-name    # set this to your application's name
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '10.x'                     # set this to the node version to use (supports 8.x, 10.x, 12.x)

jobs:
  build-and-deploy:
    runs-on: windows-latest
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: 'Login via Azure CLI'
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Setup Node ${{ env.NODE_VERSION }} Environment
      uses: actions/setup-node@v1
      with:
        node-version: ${{ env.NODE_VERSION }}

    - name: 'Resolve Project Dependencies Using Npm'
      shell: pwsh
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        npm install
        npm run build --if-present
        npm run test --if-present
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: ${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}

     - name: logout
        run: |
          az logout
```


---

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [详细了解 Azure 和 GitHub 集成](https://docs.microsoft.com/azure/developer/github/)

