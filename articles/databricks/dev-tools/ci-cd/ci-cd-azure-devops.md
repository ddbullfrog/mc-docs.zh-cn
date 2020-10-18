---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/10/2020
title: 使用 Azure DevOps 在 Azure Databricks 上进行持续集成和交付 - Azure Databricks
description: 了解如何使用 Azure DevOps 为 Azure Databricks 项目启用 CI/CD。
ms.openlocfilehash: 1731aa75313c52c70b17d98bf997d9187cb12565
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937650"
---
# <a name="continuous-integration-and-delivery-on-azure-databricks-using-azure-devops"></a>使用 Azure DevOps 在 Azure Databricks 上进行持续集成和交付

持续集成和持续交付 (CI/CD) 指的是，通过使用自动化管道，在较短且频繁的周期中开发和交付软件的过程。 虽然它已在传统的软件工程中广泛使用了数十年，并非是一种全新的过程，但对于数据工程和数据科学团队来说，它是一个越来越必要的过程。 为了使数据产品富有价值，必须及时交付这些产品。
此外，使用者必须对这些产品中的结果有效性有信心。 与仍在许多数据工程和数据科学团队中普遍使用的、手动程度更高的过程相比，通过自动执行代码的生成、测试和部署，开发团队能够更频繁、更可靠地交付发布版本。

持续集成以让你按某个频率将代码提交到源代码存储库中的分支的做法为开头。 然后，每个提交将与其他开发人员的提交合并，以确保未引入任何冲突。 通过创建生成并针对该生成运行自动测试，更改会被进一步验证。 此过程最终会产生一个项目或部署包，该项目或部署包最终会部署到目标环境（在本例中为 Azure Databricks 工作区）。

## <a name="overview-of-a-typical-azure-databricks-cicd-pipeline"></a>典型的 Azure Databricks CI/CD 管道概述

尽管它会因需求而异，但 Azure Databricks 管道的典型配置包括以下步骤：

持续集成：

1. 代码
   1. 在 Azure Databricks 笔记本中或使用外部 IDE 来开发代码和单元测试。
   1. 手动运行测试。
   1. 向 git 分支提交代码和测试。
2. 构建
   1. 收集新的和已更新的代码与测试。
   1. 运行自动测试。
   1. 生成库和非笔记本 Apache Spark 代码。
3. 发布：生成发布项目。

持续交付：

1. 部署
   1. 部署笔记本。
   1. 部署库。
2. 测试：运行自动测试并报告结果。
3. 运营：以编程方式计划数据工程、分析和机器学习工作流。

## <a name="develop-and-commit-your-code"></a>开发和提交代码

设计 CI/CD 管道的首要步骤之一是决定代码提交和分支策略，以便管理新代码与已更新的代码的开发和集成，而不会对当前正在生产环境中使用的代码产生负面影响。 此决定的一部分涉及选择一个版本控制系统，用于包含你的代码并加速该代码的升级。 Azure Databricks 支持与 [GitHub 和 Bitbucket](../../notebooks/notebooks-use.md#version-control) 的集成，从而使你可以将笔记本提交到 git 存储库。

如果你的版本控制系统不属于通过直接笔记本集成受到支持的版本控制系统，或者你需要的灵活性和控制能力超过了自助式 git 集成能够提供的，则可以使用 [Databricks CLI](../cli/index.md) 导出笔记本，并从本地计算机提交它们。 此脚本应从已设置为与相应远程存储库同步的本地 git 存储库中运行。 执行时，此脚本应：

1. 签出所需的分支。
2. 从远程分支中拉取新的更改。
3. 使用 Azure Databricks 工作区 CLI 从 Azure Databricks 工作区导出笔记本。
4. 提示用户输入提交消息，或使用默认值（如果未提供）。
5. 将更新的笔记本提交到本地分支。
6. 将更改推送到远程分支。

以下脚本会执行这些步骤：

```bash
git checkout <branch>
git pull
databricks workspace export_dir --profile <profile> -o <path> ./Workspace

dt=`date '+%Y-%m-%d %H:%M:%S'`
msg_default="DB export on $dt"
read -p "Enter the commit comment [$msg_default]: " msg
msg=${msg:-$msg_default}
echo $msg

git add .
git commit -m "<commit-message>"
git push
```

如果希望在 IDE 中而不是在 Azure Databricks 笔记本中进行开发，则可以使用新式 IDE 中内置的 VCS 集成功能或 git CLI 来提交代码。

Azure Databricks 提供了 [Databricks Connect](../databricks-connect.md)（一个用于将 IDE 连接到 Azure Databricks 群集的 SDK）。 当开发库时，此功能特别有用，因为它使你可以在 Azure Databricks 群集上运行代码并对代码进行单元测试，而无需部署该代码。 请参阅 Databricks Connect [限制](../databricks-connect.md#limitations)，确保你的用例受支持。

CI/CD 管道启动生成的时间点会有所不同，具体取决于你的分支策略和升级过程。 但是，来自各种参与者的已提交代码最终会合并到要生成并部署的指定分支。 分支管理步骤使用版本控制系统提供的接口，在 Azure Databricks 之外运行。

有大量的 CI/CD 工具可用于管理和执行管道。 本文说明了如何使用 Azure DevOps 自动化服务器。 CI/CD 是一种设计模式，因此在其他各工具中使用本文所概述的步骤和阶段时，应当对相应的管道定义语言进行一些更改。 此外，此示例管道中的大部分代码都运行标准 Python 代码（可以在其他工具中调用）。

有关将 Jenkins 与 Azure Databricks 结合使用的信息，请参阅[在 Azure Databricks 上使用 Jenkins 进行持续集成和交付](ci-cd-jenkins.md)。

## <a name="define-your-build-pipeline"></a>定义生成管道

Azure DevOps 提供一个云托管界面，用于使用 YAML 定义 CI/CD 管道的阶段。
可以在“管道”界面中定义生成管道，该管道将运行单元测试并生成部署项目。
然后，若要将代码部署到 Azure Databricks 工作区，请在发布管道中指定此部署项目。

在 Azure DevOps 项目中，打开“管道”菜单，然后单击“管道”。

> [!div class="mx-imgBorder"]
> ![Azure DevOps -“管道”菜单](../../_static/images/ci-cd/ci-cd-demo-pipelines.png)

单击“新建管道”按钮以打开“管道”编辑器，通过该编辑器可以在 `azure-pipelines.yml` 文件中定义生成。

可以使用 Git 分支选择器 ![_](../../_static/images/ci-cd/git-selector.png) 为 Git 存储库中的每个分支自定义生成过程。

> [!div class="mx-imgBorder"]
> ![Azure DevOps -“管道”编辑器](../../_static/images/ci-cd/script-in-UI.png)

`azure-pipelines.yml` 文件默认会存储在管道的 git 存储库的根目录中。 可以使用“变量”按钮配置管道引用的环境变量。

> [!div class="mx-imgBorder"]
> ![Azure DevOps -“变量”按钮](../../_static/images/ci-cd/variables-button.png)

有关 Azure DevOps 和生成管道的详细信息，请参阅 [Azure DevOps 文档](https://docs.microsoft.com/azure/devops/)。

### <a name="configure-your-build-agent"></a>配置生成代理

为了执行管道，Azure DevOps 提供了云托管的按需执行代理，这些代理支持部署到 Kubernetes、VM、Azure Functions、Azure Web 应用等目标。 在此示例中，将使用按需代理自动将代码部署到目标 Azure Databricks 工作区。 在执行时，管道所需的工具或包必须已在管道脚本中定义，并已安装在代理上。

此示例需要以下依赖项：

* Conda - Conda 是一种开源环境管理系统
* Python v 3.7.3 - Python 将用于运行测试、生成部署 wheel 和执行部署脚本。 Python 的版本非常重要，因为测试要求代理上运行的 Python 版本应与 Azure Databricks 群集的 Python 版本相匹配。 此示例使用 Databricks Runtime 6.4，其中包含 Python 3.7。
* Python 库：`requests`、`databricks-connect`、`databricks-cli`、`pytest`

此处提供了一个示例管道 (`azure-pipelines.yml`)。 完整脚本如下。 本文逐步介绍该脚本的每个部分。

```yaml
# Azure Databricks Build Pipeline
# azure-pipelines.yml

trigger:
- release

pool:
  name: Hosted Ubuntu 1604

steps:
- task: UsePythonVersion@0
  displayName: 'Use Python 3.7'
  inputs:
    versionSpec: 3.7

- script: |
    pip install pytest requests setuptools wheel
    pip install -U databricks-connect==6.4.*
  displayName: 'Load Python Dependencies'

- script: |
    echo "y
    $(WORKSPACE-REGION-URL)
    $(CSE-DEVELOP-PAT)
    $(EXISTING-CLUSTER-ID)
    $(WORKSPACE-ORG-ID)
    15001" | databricks-connect configure
  displayName: 'Configure DBConnect'

- checkout: self
  persistCredentials: true
  clean: true

- script: git checkout release
  displayName: 'Get Latest Branch'

- script: |
    python -m pytest --junit-xml=$(Build.Repository.LocalPath)/logs/TEST-LOCAL.xml
$(Build.Repository.LocalPath)/libraries/python/dbxdemo/test*.py || true

  displayName: 'Run Python Unit Tests for library code'

- task: PublishTestResults@2
  inputs:
    testResultsFiles: '**/TEST-*.xml'
    failTaskOnFailedTests: true
    publishRunAttachments: true

- script: |
    cd $(Build.Repository.LocalPath)/libraries/python/dbxdemo
    python3 setup.py sdist bdist_wheel
    ls dist/
  displayName: 'Build Python Wheel for Libs'

- script: |
    git diff --name-only --diff-filter=AMR HEAD^1 HEAD | xargs -I '{}' cp --parents -r '{}' $(Build.BinariesDirectory)

    mkdir -p $(Build.BinariesDirectory)/libraries/python/libs
    cp $(Build.Repository.LocalPath)/libraries/python/dbxdemo/dist/*.* $(Build.BinariesDirectory)/libraries/python/libs

    mkdir -p $(Build.BinariesDirectory)/cicd-scripts
    cp $(Build.Repository.LocalPath)/cicd-scripts/*.* $(Build.BinariesDirectory)/cicd-scripts

  displayName: 'Get Changes'

- task: ArchiveFiles@2
  inputs:
    rootFolderOrFile: '$(Build.BinariesDirectory)'
    includeRootFolder: false
    archiveType: 'zip'
    archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'
    replaceExistingArchive: true

- task: PublishBuildArtifacts@1
  inputs:
    ArtifactName: 'DatabricksBuild'
```

### <a name="set-up-the-pipeline"></a><a id="set-up-pipeline"> </a><a id="set-up-the-pipeline"> </a>设置管道

在设置阶段，你将配置生成代理、Databricks CLI、Databricks Connect 及其连接信息。

```yaml
# Specify the trigger event to start the build pipeline.
# In this case, new code merged into the release branch initiates a new build.
trigger:
- release

# Specify the OS for the agent
pool:
  name: Hosted Ubuntu 1604

# Install Python. The version must match the version on the Databricks cluster.
steps:
- task: UsePythonVersion@0
  displayName: 'Use Python 3.7'
  inputs:
    versionSpec: 3.7

# Install required Python modules, including databricks-connect, required to execute a unit test
# on a cluster.
- script: |
    pip install pytest requests setuptools wheel
    pip install -U databricks-connect==6.4.*
  displayName: 'Load Python Dependencies'

# Use environment variables to pass Databricks login information to the Databricks Connect
# configuration function
- script: |
    echo "y
    $(WORKSPACE-REGION-URL)
    $(CSE-DEVELOP-PAT)
    $(EXISTING-CLUSTER-ID)
    $(WORKSPACE-ORG-ID)
    15001" | databricks-connect configure
  displayName: 'Configure DBConnect'
```

### <a name="get-the-latest-changes"></a>获取最新更改

此阶段会将指定分支中的代码下载到“代理执行”代理。

```yaml
- checkout: self
  persistCredentials: true
  clean: true

- script: git checkout release
  displayName: 'Get Latest Branch'
```

### <a name="unit-tests-in-azure-databricks-notebooks"></a>Azure Databricks 笔记本中的单元测试

对于在 Azure Databricks 笔记本外开发的库代码，此过程类似于传统的软件开发做法。 你将使用测试框架（如 Python `pytest` 模块）编写单元测试，并使用 JUnit 格式的 XML 文件来存储测试结果。

Azure Databricks 代码是用于在 Azure Databricks 群集上执行的 Apache Spark 代码。 若要对此代码进行单元测试，可以使用在[设置管道](#set-up-pipeline)中配置的 Databricks Connect SDK。

### <a name="test-library-code-using-databricks-connect"></a>使用 Databricks Connect 测试库代码

管道的此阶段调用单元测试，并为测试和输出文件指定名称和位置。

```yaml
- script: |
    python -m pytest --junit-xml=$(Build.Repository.LocalPath)/logs/TEST-LOCAL.xml $(Build.Repository.LocalPath)/libraries/python/dbxdemo/test*.py || true
    ls logs
  displayName: 'Run Python Unit Tests for library code'
```

以下代码片段 (`addcol.py`) 是可能安装在 Azure Databricks 群集上的库函数。 此简单函数会将由文本填充的新列添加到 Apache Spark 数据帧中。

```python
# addcol.py
import pyspark.sql.functions as F

def with_status(df):
    return df.withColumn("status", F.lit("checked"))
```

以下测试 `test-addcol.py` 将一个模拟数据帧对象传递到 `with_status` 函数，该函数在 [addcol.py](#addcol-py) 中定义。 然后，结果将与一个包含预期值的数据帧对象进行比较。 如果值匹配，则测试通过。

```python
# test-addcol.py
import pytest

from dbxdemo.spark import get_spark
from dbxdemo.appendcol import with_status

class TestAppendCol(object):

    def test_with_status(self):
        source_data = [
            ("pete", "pan", "peter.pan@databricks.com"),
            ("jason", "argonaut", "jason.argonaut@databricks.com")
        ]
        source_df = get_spark().createDataFrame(
            source_data,
            ["first_name", "last_name", "email"]
        )

        actual_df = with_status(source_df)

        expected_data = [
            ("pete", "pan", "peter.pan@databricks.com", "checked"),
            ("jason", "argonaut", "jason.argonaut@databricks.com", "checked")
        ]
        expected_df = get_spark().createDataFrame(
            expected_data,
            ["first_name", "last_name", "email", "status"]
        )

        assert(expected_df.collect() == actual_df.collect())
```

### <a name="package-library-code"></a><a id="addcol-py"> </a><a id="package-library-code"> </a>将库代码打包

管道的此阶段将库代码打包到一个 Python wheel 中。

```yaml
- script: |
    cd $(Build.Repository.LocalPath)/libraries/python/dbxdemo
    python3 setup.py sdist bdist_wheel
    ls dist/
  displayName: 'Build Python Wheel for Libs'
```

### <a name="publish-test-results"></a>发布测试结果

执行所有单元测试后，将结果发布到 Azure DevOps。 这使你可以将与生成过程的状态相关的报表和仪表板可视化。

```yaml
- task: PublishTestResults@2
  inputs:
    testResultsFiles: '**/TEST-*.xml'
    failTaskOnFailedTests: true
    publishRunAttachments: true
```

### <a name="generate-and-store-a-deployment-artifact"></a>生成并存储部署项目

生成管道的最后一步是生成部署项目。 为此，你需要收集所有要部署到 Azure Databricks 环境的新代码或已更新的代码，包括要部署到工作区的笔记本代码、由生成过程生成的 `.whl` 库，以及用于存档的测试结果摘要。

```yaml
# Use git diff to flag files added in the most recent git merge
- script: |
    git diff --name-only --diff-filter=AMR HEAD^1 HEAD | xargs -I '{}' cp --parents -r '{}' $(Build.BinariesDirectory)

# Add the wheel file you just created along with utility scripts used by the Release pipeline
# The implementation in your Pipeline may be different.
# The objective is to add all files intended for the current release.
    mkdir -p $(Build.BinariesDirectory)/libraries/python/libs
    cp $(Build.Repository.LocalPath)/libraries/python/dbxdemo/dist/*.* $(Build.BinariesDirectory)/libraries/python/libs

    mkdir -p $(Build.BinariesDirectory)/cicd-scripts
    cp $(Build.Repository.LocalPath)/cicd-scripts/*.* $(Build.BinariesDirectory)/cicd-scripts
  displayName: 'Get Changes'

# Create the deployment artifact and publish it to the artifact repository
- task: ArchiveFiles@2
  inputs:
    rootFolderOrFile: '$(Build.BinariesDirectory)'
    includeRootFolder: false
    archiveType: 'zip'
    archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'
    replaceExistingArchive: true

- task: PublishBuildArtifacts@1
  inputs:
    ArtifactName: 'DatabricksBuild'
```

## <a name="define-your-release-pipeline"></a>定义发布管道

发布管道将项目部署到 Azure Databricks 环境。 通过将发布管道与生成管道分离，可以创建一个生成而无需部署它，或者一次部署来自多个生成的项目。

1. 在 Azure DevOps 项目中，转到“管道”菜单，然后单击“发布”。

   > [!div class="mx-imgBorder"]
   > ![Azure DevOps - 发布](../../_static/images/ci-cd/ci-cd-releases.png)

2. 在屏幕右侧是为常见部署模式特别推荐的模板的列表。 对于此管道，请单击 ![_](../../_static/images/ci-cd/empty-job.png).

   > [!div class="mx-imgBorder"]
   > ![Azure DevOps - 发布管道 1](../../_static/images/ci-cd/release-pipeline-1.png)

3. 在屏幕左侧的“项目”框中，单击 ![_](../../_static/images/ci-cd/plus-add.png) 并选择先前创建的生成管道。

   > [!div class="mx-imgBorder"]
   > ![Azure DevOps 发布管道 2](../../_static/images/ci-cd/release-pipeline-add-artifact.png)

可以通过单击 ![_](../../_static/images/ci-cd/lightning-bolt.png)（用于在屏幕右侧显示触发选项），配置该管道的触发方式。 如果希望根据生成项目的可用性或在某个拉取请求工作流后自动启动发布，请启用适当的触发器。

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 发布管道 - 阶段 1](../../_static/images/ci-cd/release-pipeline-stage-1.png)

若要为部署添加步骤或任务，请单击阶段对象内的链接。

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 发布管道 - 添加阶段](../../_static/images/ci-cd/stage-button.png)

### <a name="add-tasks"></a>添加任务

若要添加任务，请在“代理作业”部分中单击加号，如下图中的红色箭头所指示。
此时将显示可用任务的可搜索列表。 还有一个市场，其中提供可用于补充标准 Azure DevOps 任务的第三方插件。

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 添加任务](../../_static/images/ci-cd/release-pipeline-add-task.png)

### <a name="set-python-version"></a>设置 Python 版本

你添加的第一个任务是“使用 Python 版本”。 与生成管道一样，你需要确保 Python 版本与后续任务中调用的脚本兼容。

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 设置 python 版本 1](../../_static/images/ci-cd/release-pipeline-set-python-version.png)

在本例中，将 Python 版本设置为 3.7。

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 设置 python 版本 2](../../_static/images/ci-cd/use-python-version.png)

### <a name="unpackage-the-build-artifact"></a>将生成项目解包

使用“提取文件”任务提取存档。 将“存档文件模式”设置为 `*.zip`，并将“目标文件夹”设置为系统变量“$(agent.builddirectory)”。
可以选择性地设置显示名称；这是将出现在屏幕上的“代理作业”下的名称。

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 解包](../../_static/images/ci-cd/unpackage-build-artifact.png)

### <a name="deploy-the-notebooks-to-the-workspace"></a>将笔记本部署到工作区

为了部署笔记本，此示例使用由 Data Thirst 开发的第三方任务“Databricks 部署笔记本”。

* 输入环境变量，以设置“Azure 区域”的值和“Databricks 持有者令牌”的值。
* 将“源文件路径”设置为包含笔记本的已提取目录路径。
* 将“目标文件路径”设置为 Azure Databricks 工作区目录结构中的所需路径。

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 部署笔记本](../../_static/images/ci-cd/databricks-deploy-notebooks.png)

### <a name="deploy-the-library-to-dbfs"></a>将库部署到 DBFS

若要部署 Python `*.war` 文件，请使用同样由 Data Thirst 开发的第三方任务“Databricks 文件到 DBFS”。

* 输入环境变量，以设置“Azure 区域”的值和“Databricks 持有者令牌”的值。
* 将“本地根文件夹”设置为包含 Python 库的已提取目录路径。
* 将“DBFS 中的目标文件夹”设置为所需的 DBFS 路径。

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 文件部署](../../_static/images/ci-cd/dbfs-file-deploy.png)

### <a name="install-the-library-on-a-cluster"></a>将库安装到群集上

最后的代码部署任务会将库安装到特定群集上。 为此，请创建“Python 脚本”任务。
Python 脚本 `installWhlLibrary.py` 位于由我们的生成管道创建的项目中。

* 将“脚本路径”设置为 `$(agent.builddirectory)/cicd-scripts/installWhlLibrary.py`。 `installWhlLibrary.py` 脚本采用五个参数：
  * `shard` - 目标工作区的 URL（例如 `https://<region>.databricks.azure.cn`）
  * `token` - 用于工作区的个人访问令牌
  * `clusterid` -要将库安装到的群集的 ID
  * `libs` - 已提取的目录，其中包含库
  * `dbfspath` - 用于检索库的 DBFS 文件系统中的路径

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 安装库](../../_static/images/ci-cd/python-script-install-library.png)

在 Azure Databricks 群集上安装新版本的库之前，必须先卸载现有库。
为此，请在 Python 脚本中调用 Databricks REST API 以执行以下步骤：

1. 检查是否已安装该库。
2. 卸载该库。
3. 如果执行了任何卸载，请重新启动群集。
4. 请等到群集再次运行，然后继续操作。
5. 安装该库。

```python
# installWhlLibrary.py
#!/usr/bin/python3
import json
import requests
import sys
import getopt
import time
import os

def main():
    shard = ''
    token = ''
    clusterid = ''
    libspath = ''
    dbfspath = ''

    try:
        opts, args = getopt.getopt(sys.argv[1:], 'hstcld',
                                   ['shard=', 'token=', 'clusterid=', 'libs=', 'dbfspath='])
    except getopt.GetoptError:
        print(
            'installWhlLibrary.py -s <shard> -t <token> -c <clusterid> -l <libs> -d <dbfspath>')
        sys.exit(2)

    for opt, arg in opts:
        if opt == '-h':
            print(
                'installWhlLibrary.py -s <shard> -t <token> -c <clusterid> -l <libs> -d <dbfspath>')
            sys.exit()
        elif opt in ('-s', '--shard'):
            shard = arg
        elif opt in ('-t', '--token'):
            token = arg
        elif opt in ('-c', '--clusterid'):
            clusterid = arg
        elif opt in ('-l', '--libs'):
            libspath=arg
        elif opt in ('-d', '--dbfspath'):
            dbfspath=arg

    print('-s is ' + shard)
    print('-t is ' + token)
    print('-c is ' + clusterid)
    print('-l is ' + libspath)
    print('-d is ' + dbfspath)

    # Uninstall library if exists on cluster
    i=0

    # Generate array from walking local path
    libslist = []
    for path, subdirs, files in os.walk(libspath):
        for name in files:

            name, file_extension = os.path.splitext(name)
            if file_extension.lower() in ['.whl']:
                libslist.append(name + file_extension.lower())

    for lib in libslist:
        dbfslib = dbfspath + '/' + lib
        print(dbfslib + ' before:' + getLibStatus(shard, token, clusterid, dbfslib))

        if (getLibStatus(shard, token, clusterid, dbfslib) != 'not found'):
            print(dbfslib + " exists. Uninstalling.")
            i = i + 1
            values = {'cluster_id': clusterid, 'libraries': [{'whl': dbfslib}]}

            resp = requests.post(shard + '/api/2.0/libraries/uninstall', data=json.dumps(values), auth=("token", token))
            runjson = resp.text
            d = json.loads(runjson)
            print(dbfslib + ' after:' + getLibStatus(shard, token, clusterid, dbfslib))

            # Restart if libraries uninstalled
            if i > 0:
                values = {'cluster_id': clusterid}
                print("Restarting cluster:" + clusterid)
                resp = requests.post(shard + '/api/2.0/clusters/restart', data=json.dumps(values), auth=("token", token))
                restartjson = resp.text
                print(restartjson)

                p = 0
                waiting = True
                while waiting:
                    time.sleep(30)
                    clusterresp = requests.get(shard + '/api/2.0/clusters/get?cluster_id=' + clusterid,
                                           auth=("token", token))
                    clusterjson = clusterresp.text
                    jsonout = json.loads(clusterjson)
                    current_state = jsonout['state']
                    print(clusterid + " state:" + current_state)
                    if current_state in ['TERMINATED', 'RUNNING','INTERNAL_ERROR', 'SKIPPED'] or p >= 10:
                        break
                    p = p + 1

        print("Installing " + dbfslib)
        values = {'cluster_id': clusterid, 'libraries': [{'whl': 'dbfs:' + dbfslib}]}

        resp = requests.post(shard + '/api/2.0/libraries/install', data=json.dumps(values), auth=("token", token))
        runjson = resp.text
        d = json.loads(runjson)
        print(dbfslib + ' after:' + getLibStatus(shard, token, clusterid, dbfslib))

def getLibStatus(shard, token, clusterid, dbfslib):

    resp = requests.get(shard + '/api/2.0/libraries/cluster-status?cluster_id='+ clusterid, auth=("token", token))
    libjson = resp.text
    d = json.loads(libjson)
    if (d.get('library_statuses')):
        statuses = d['library_statuses']

        for status in statuses:
            if (status['library'].get('whl')):
                if (status['library']['whl'] == 'dbfs:' + dbfslib):
                    return status['status']
                else:
                    return "not found"
    else:
        # No libraries found
        return "not found"

if __name__ == '__main__':
    main()
```

### <a name="run-integration-tests-from-an-azure-databricks-notebook"></a>从 Azure Databricks 笔记本运行集成测试

还可以直接从包含断言的笔记本运行测试。 在本例中，可以使用已在单元测试中使用的同一测试，但现在它会从刚刚安装在群集上的 `whl` 中导入已安装的 `appendcol` 库。

若要自动执行此测试，并将其包括在 CI/CD 管道中，请使用 Databricks REST API 从 CI/CD 服务器执行笔记本。 这样，你便可以使用 `pytest` 来检查笔记本执行是成功还是失败。 任何断言失败都会出现在 REST API 返回的 JSON 输出和 JUnit 测试结果中。

#### <a name="step-1-configure-the-test-environment"></a>步骤 1：配置测试环境

按下面所示创建“命令行”任务。 此任务包括用于为笔记本执行日志和测试摘要创建目录的命令。 它还包括一个 pip 命令，用于安装所需的 `pytest` 和 `requests` 模块。

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 配置测试环境](../../_static/images/ci-cd/commandline-configure-test-envt.png)

#### <a name="step-2-run-the-notebook"></a>步骤 2：运行笔记本

创建“Python 脚本”任务，并对其进行配置，如下所示：

* 将“脚本路径”设置为 `$(agent.builddirectory)/cicd-scripts/executeNotebook.py`。 此脚本采用六个参数：
  * `shard` - 目标工作区的 URL（例如 [https://chinaeast2.databricks.azure.cn](https://chinaeast2.databricks.azure.cn)）
  * `token` - 用于工作区的个人访问令牌
  * `clusterid` - 要执行该测试的群集的 ID
  * `localpath` - 已提取的目录，其中包含测试笔记本
  * `workspacepath` - 已部署了测试笔记本的工作区中的路径
  * `outfilepath` - 你已创建的用于存储由 REST API 返回的 JSON 输出的路径

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 执行笔记本](../../_static/images/ci-cd/pythonscript-executenotebooks.png)

脚本 `executenotebook.py` 使用用于提交匿名作业的“作业运行提交”终结点来运行此笔记本。 由于此终结点是异步的，因此它使用 REST 调用最初返回的作业 ID 来轮询作业的状态。 作业完成后，JSON 输出会保存到在调用时传递的函数参数所指定的路径。

```python
# executenotebook.py
#!/usr/bin/python3
import json
import requests
import os
import sys
import getopt
import time

def main():
    shard = ''
    token = ''
    clusterid = ''
    localpath = ''
    workspacepath = ''
    outfilepath = ''

    try:
        opts, args = getopt.getopt(sys.argv[1:], 'hs:t:c:lwo',
                                   ['shard=', 'token=', 'clusterid=', 'localpath=', 'workspacepath=', 'outfilepath='])
    except getopt.GetoptError:
        print(
            'executenotebook.py -s <shard> -t <token>  -c <clusterid> -l <localpath> -w <workspacepath> -o <outfilepath>)')
        sys.exit(2)

    for opt, arg in opts:
        if opt == '-h':
            print(
                'executenotebook.py -s <shard> -t <token> -c <clusterid> -l <localpath> -w <workspacepath> -o <outfilepath>')
            sys.exit()
        elif opt in ('-s', '--shard'):
            shard = arg
        elif opt in ('-t', '--token'):
            token = arg
        elif opt in ('-c', '--clusterid'):
            clusterid = arg
        elif opt in ('-l', '--localpath'):
            localpath = arg
        elif opt in ('-w', '--workspacepath'):
            workspacepath = arg
        elif opt in ('-o', '--outfilepath'):
            outfilepath = arg

    print('-s is ' + shard)
    print('-t is ' + token)
    print('-c is ' + clusterid)
    print('-l is ' + localpath)
    print('-w is ' + workspacepath)
    print('-o is ' + outfilepath)
    # Generate array from walking local path

    notebooks = []
    for path, subdirs, files in os.walk(localpath):
        for name in files:
            fullpath = path + '/' + name
            # removes localpath to repo but keeps workspace path
            fullworkspacepath = workspacepath + path.replace(localpath, '')

            name, file_extension = os.path.splitext(fullpath)
            if file_extension.lower() in ['.scala', '.sql', '.r', '.py']:
                row = [fullpath, fullworkspacepath, 1]
                notebooks.append(row)

    # run each element in array
    for notebook in notebooks:
        nameonly = os.path.basename(notebook[0])
        workspacepath = notebook[1]

        name, file_extension = os.path.splitext(nameonly)

        # workpath removes extension
        fullworkspacepath = workspacepath + '/' + name

        print('Running job for:' + fullworkspacepath)
        values = {'run_name': name, 'existing_cluster_id': clusterid, 'timeout_seconds': 3600, 'notebook_task': {'notebook_path': fullworkspacepath}}

        resp = requests.post(shard + '/api/2.0/jobs/runs/submit',
                             data=json.dumps(values), auth=("token", token))
        runjson = resp.text
        print("runjson:" + runjson)
        d = json.loads(runjson)
        runid = d['run_id']

        i=0
        waiting = True
        while waiting:
            time.sleep(10)
            jobresp = requests.get(shard + '/api/2.0/jobs/runs/get?run_id='+str(runid),
                             data=json.dumps(values), auth=("token", token))
            jobjson = jobresp.text
            print("jobjson:" + jobjson)
            j = json.loads(jobjson)
            current_state = j['state']['life_cycle_state']
            runid = j['run_id']
            if current_state in ['TERMINATED', 'INTERNAL_ERROR', 'SKIPPED'] or i >= 12:
                break
            i=i+1

        if outfilepath != '':
            file = open(outfilepath + '/' +  str(runid) + '.json', 'w')
            file.write(json.dumps(j))
            file.close()

if __name__ == '__main__':
    main()
```

#### <a name="step-3-generate-and-evaluate-test-results"></a>步骤 3：生成和评估测试结果

此任务使用 `pytest` 执行 Python 脚本，以确定测试笔记本中的断言是已成功还是已失败。

创建“命令行”任务。 “脚本”应为：

```bash
python -m pytest --junit-xml=$(agent.builddirectory)\logs\xml\TEST-notebookout.xml --jsonpath=$(agent.builddirectory)\logs\json\ $(agent.builddirectory)\cicd-scripts\evaluatenotebookruns.py || true
```

参数包括：

* `junit-xml` - 要在其中生成 JUnit 测试摘要日志的路径
* `jsonpath` - 你已创建的用于存储由 REST API 返回的 JSON 输出的路径

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 生成测试结果](../../_static/images/ci-cd/commandline-generate-test-results.png)

脚本 `evaluatenotebookruns.py` 定义了 `test_job_run` 函数，该函数分析并评估由上一任务生成的 JSON。
另一个测试 `test_performance` 会查找运行时间长于预期时间的测试。

```python
# evaluatenotebookruns.py
import unittest
import json
import glob
import os

class TestJobOutput(unittest.TestCase):

    test_output_path = '#ENV#'

    def test_performance(self):
        path = self.test_output_path
        statuses = []

        for filename in glob.glob(os.path.join(path, '*.json')):
            print('Evaluating: ' + filename)
            data = json.load(open(filename))
            duration = data['execution_duration']
            if duration > 100000:
                status = 'FAILED'
            else:
                status = 'SUCCESS'

            statuses.append(status)

        self.assertFalse('FAILED' in statuses)

    def test_job_run(self):
        path = self.test_output_path
        statuses = []

        for filename in glob.glob(os.path.join(path, '*.json')):
            print('Evaluating: ' + filename)
            data = json.load(open(filename))
            status = data['state']['result_state']
            statuses.append(status)

        self.assertFalse('FAILED' in statuses)

if __name__ == '__main__':
    unittest.main()
```

### <a name="publish-test-results"></a>发布测试结果

使用“发布测试结果”任务可将 JSON 结果存档，并将测试结果发布到 Azure DevOps 测试中心。 这使你可以将与测试运行的状态相关的报表和仪表板可视化。

> [!div class="mx-imgBorder"]
> ![Azure DevOps - 发布测试结果](../../_static/images/ci-cd/publish-test-results.png)

此时，你已使用 CI/CD 管道完成了一个集成和部署周期。 通过自动执行此过程，可以确保代码通过有效、一致且可重复的过程进行测试和部署。