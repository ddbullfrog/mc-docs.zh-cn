---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 06/03/2020
title: 在 Azure Databricks 上使用 Jenkins 进行持续集成和交付 - Azure Databricks
description: 了解如何使用 Jenkins 为 Azure Databricks 项目启用 CI/CD。
ms.openlocfilehash: b1d306679d38bc34c1359ac9aed480c67da45e61
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937642"
---
# <a name="continuous-integration-and-delivery-on-azure-databricks-using-jenkins"></a>在 Azure Databricks 上使用 Jenkins 进行持续集成和交付

持续集成和持续交付 (CI/CD) 指的是，通过使用自动化管道，在较短且频繁的周期中开发和交付软件的过程。 虽然它已在传统的软件工程中广泛使用了数十年，并非是一种全新的过程，但对于数据工程和数据科学团队来说，它是一个越来越必要的过程。 为了使数据产品富有价值，必须及时交付这些产品。
此外，使用者必须对这些产品中的结果有效性有信心。 与仍在许多数据工程和数据科学团队中普遍使用的、手动程度更高的过程相比，通过自动执行代码的生成、测试和部署，开发团队能够更频繁、更可靠地交付发布版本。

持续集成始于让你按某个频率将代码提交到源代码存储库中的分支的做法。 然后，每个提交将与其他开发人员的提交合并，以确保未引入任何冲突。 通过创建生成并针对该生成运行自动测试，更改会被进一步验证。 此过程最终会产生一个项目或部署包，该项目或部署包最终会部署到目标环境（在本例中为 Azure Databricks 工作区）。

## <a name="overview-of-a-typical-azure-databricks-cicd-pipeline"></a>典型的 Azure Databricks CI/CD 管道概述

尽管它可以因需求而异，但 Azure Databricks 管道的典型配置包括以下步骤：

**持续集成：**

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
3. 使用 Databricks CLI 从 Azure Databricks 工作区导出笔记本。
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

CI/CD 管道启动生成的时间点将因分支策略和升级过程而异。 但是，来自各种参与者的已提交代码最终会合并到要生成并部署的指定分支。 分支管理步骤使用版本控制系统提供的接口，在 Azure Databricks 之外运行。

有大量的 CI/CD 工具可用于管理和执行管道。 本文说明了如何使用 [Jenkins](https://jenkins.io/) 自动化服务器。 CI/CD 是一种设计模式，因此在其他各工具中使用本文所概述的步骤和阶段时，应当对相应的管道定义语言进行一些更改。 此外，此示例管道中的大部分代码都运行标准 Python 代码（可以在其他工具中调用）。

有关将 Azure DevOps 与 Azure Databricks 结合使用的信息，请参阅[在 Azure Databricks 上使用 Azure DevOps 进行持续集成和交付](ci-cd-azure-devops.md)。

## <a name="configure-your-agent"></a>配置代理

Jenkins 使用主服务进行协调，并使用一对多执行代理。 在此示例中，将使用 Jenkins 服务器附带的默认永久代理节点。
必须在代理（在本例中为 Jenkins 服务器）上手动安装管道所需的以下工具和包：

* [Conda](https://docs.conda.io/en/latest/)：一种开源 Python 环境管理系统。
* Python 3.7.3：用于运行测试、构建部署 wheel 和执行部署脚本。 Python 的版本非常重要，因为测试要求代理上运行的 Python 版本应与 Azure Databricks 群集的 Python 版本相匹配。 此示例使用 Databricks Runtime 6.4，其中包含 Python 3.7。
* Python 库：`requests`、`databricks-connect`、`databricks-cli` 和 `pytest`。

## <a name="design-the-pipeline"></a>设计管道

Jenkins 提供了几种不同的项目类型，以用于创建 CI/CD 管道。 此示例实现 Jenkins 管道。 Jenkins 管道提供了一个接口，用于通过 [Groovy](http://groovy-lang.org/) 代码调用和配置 Jenkins 插件以定义管道中的阶段。

> [!div class="mx-imgBorder"]
> ![Jenkins 项目类型](../../_static/images/jenkins-project-types.png)

将管道定义写入文本文件（称为 Jenkinsfile）中，后者又会签入到项目的源代码管理存储库中。 有关详细信息，请参阅 [Jenkins 管道](https://jenkins.io/doc/book/pipeline/)。 此处提供了一个示例管道：

```groovy
// Jenkinsfile
node {
  def GITREPO         = "/var/lib/jenkins/workspace/${env.JOB_NAME}"
  def GITREPOREMOTE   = "https://github.com/<repo>"
  def GITHUBCREDID    = "<github-token>"
  def CURRENTRELEASE  = "<release>"
  def DBTOKEN         = "<databricks-token>"
  def DBURL           = "https://<databricks-instance>"
  def SCRIPTPATH      = "${GITREPO}/Automation/Deployments"
  def NOTEBOOKPATH    = "${GITREPO}/Workspace"
  def LIBRARYPATH     = "${GITREPO}/Libraries"
  def BUILDPATH       = "${GITREPO}/Builds/${env.JOB_NAME}-${env.BUILD_NUMBER}"
  def OUTFILEPATH     = "${BUILDPATH}/Validation/Output"
  def TESTRESULTPATH  = "${BUILDPATH}/Validation/reports/junit"
  def WORKSPACEPATH   = "/Shared/<path>"
  def DBFSPATH        = "dbfs:<dbfs-path>"
  def CLUSTERID       = "<cluster-id>"
  def CONDAPATH       = "<conda-path>"
  def CONDAENV        = "<conda-env>"

  stage('Setup') {
      withCredentials([string(credentialsId: DBTOKEN, variable: 'TOKEN')]) {
        sh """#!/bin/bash
            # Configure Conda environment for deployment & testing
            source ${CONDAPATH}/bin/activate ${CONDAENV}

            # Configure Databricks CLI for deployment
            echo "${DBURL}
            $TOKEN" | databricks configure --token

            # Configure Databricks Connect for testing
            echo "${DBURL}
            $TOKEN
            ${CLUSTERID}
            0
            15001" | databricks-connect configure
           """
      }
  }
  stage('Checkout') { // for display purposes
    echo "Pulling ${CURRENTRELEASE} Branch from Github"
    git branch: CURRENTRELEASE, credentialsId: GITHUBCREDID, url: GITREPOREMOTE
  }
  stage('Run Unit Tests') {
    try {
        sh """#!/bin/bash

              # Enable Conda environment for tests
              source ${CONDAPATH}/bin/activate ${CONDAENV}

              # Python tests for libs
              python3 -m pytest --junit-xml=${TESTRESULTPATH}/TEST-libout.xml ${LIBRARYPATH}/python/dbxdemo/test*.py || true
           """
    } catch(err) {
      step([$class: 'JUnitResultArchiver', testResults: '--junit-xml=${TESTRESULTPATH}/TEST-*.xml'])
      if (currentBuild.result == 'UNSTABLE')
        currentBuild.result = 'FAILURE'
      throw err
    }
  }
  stage('Package') {
    sh """#!/bin/bash

          # Enable Conda environment for tests
          source ${CONDAPATH}/bin/activate ${CONDAENV}

          # Package Python library to wheel
          cd ${LIBRARYPATH}/python/dbxdemo
          python3 setup.py sdist bdist_wheel
       """
  }
  stage('Build Artifact') {
    sh """mkdir -p ${BUILDPATH}/Workspace
          mkdir -p ${BUILDPATH}/Libraries/python
          mkdir -p ${BUILDPATH}/Validation/Output
          #Get modified files
          git diff --name-only --diff-filter=AMR HEAD^1 HEAD | xargs -I '{}' cp --parents -r '{}' ${BUILDPATH}

          # Get packaged libs
          find ${LIBRARYPATH} -name '*.whl' | xargs -I '{}' cp '{}' ${BUILDPATH}/Libraries/python/

          # Generate artifact
          tar -czvf Builds/latest_build.tar.gz ${BUILDPATH}
       """
    archiveArtifacts artifacts: 'Builds/latest_build.tar.gz'
  }
  stage('Deploy') {
    sh """#!/bin/bash
          # Enable Conda environment for tests
          source ${CONDAPATH}/bin/activate ${CONDAENV}

          # Use Databricks CLI to deploy notebooks
          databricks workspace import_dir ${BUILDPATH}/Workspace ${WORKSPACEPATH}

          dbfs cp -r ${BUILDPATH}/Libraries/python ${DBFSPATH}
       """
    withCredentials([string(credentialsId: DBTOKEN, variable: 'TOKEN')]) {
        sh """#!/bin/bash

              #Get space delimited list of libraries
              LIBS=\$(find ${BUILDPATH}/Libraries/python/ -name '*.whl' | sed 's#.*/##' | paste -sd " ")

              #Script to uninstall, reboot if needed & instsall library
              python3 ${SCRIPTPATH}/installWhlLibrary.py --workspace=${DBURL}\
                        --token=$TOKEN\
                        --clusterid=${CLUSTERID}\
                        --libs=\$LIBS\
                        --dbfspath=${DBFSPATH}
           """
    }
  }
  stage('Run Integration Tests') {
    withCredentials([string(credentialsId: DBTOKEN, variable: 'TOKEN')]) {
        sh """python3 ${SCRIPTPATH}/executenotebook.py --workspace=${DBURL}\
                        --token=$TOKEN\
                        --clusterid=${CLUSTERID}\
                        --localpath=${NOTEBOOKPATH}/VALIDATION\
                        --workspacepath=${WORKSPACEPATH}/VALIDATION\
                        --outfilepath=${OUTFILEPATH}
           """
    }
    sh """sed -i -e 's #ENV# ${OUTFILEPATH} g' ${SCRIPTPATH}/evaluatenotebookruns.py
          python3 -m pytest --junit-xml=${TESTRESULTPATH}/TEST-notebookout.xml ${SCRIPTPATH}/evaluatenotebookruns.py || true
       """
  }
  stage('Report Test Results') {
    sh """find ${OUTFILEPATH} -name '*.json' -exec gzip --verbose {} \\;
          touch ${TESTRESULTPATH}/TEST-*.xml
       """
    junit "**/reports/junit/*.xml"
  }
}
```

本文的余下内容讨论管道中的每个步骤。

## <a name="define-environment-variables"></a>定义环境变量

可以定义环境变量，以便在不同的管道中使用管道阶段。

* `GITREPO`：git 存储库根目录的本地路径
* `GITREPOREMOTE`：git 存储库的 Web URL
* `GITHUBCREDID`：GitHub 个人访问令牌的 Jenkins 凭据 ID
* `CURRENTRELEASE`：部署分支
* `DBTOKEN`：Azure Databricks 个人访问令牌的 Jenkins 凭据 ID
* `DBURL`：Azure Databricks 工作区的 Web URL
* `SCRIPTPATH`：自动化脚本的 git 项目目录的本地路径
* `NOTEBOOKPATH`：笔记本的 git 项目目录的本地路径
* `LIBRARYPATH`：库代码或其他 DBFS 代码的 git 项目目录的本地路径
* `BUILDPATH`：生成工件的目录的本地路径
* `OUTFILEPATH`：从自动测试生成的 JSON 结果文件的本地路径
* `TESTRESULTPATH`：Junit 测试结果摘要的目录的本地路径
* `WORKSPACEPATH`：笔记本的 Azure Databricks 工作区路径
* `DBFSPATH`：库和非笔记本代码的 Azure Databricks DBFS 路径
* `CLUSTERID`：用于运行测试的 Azure Databricks 群集 ID
* `CONDAPATH`：Conda 安装的路径
* `CONDAENV`：包含生成依赖项库的 Conda 环境的名称

## <a name="set-up-the-pipeline"></a>设置管道

在 `Setup` 阶段，你将使用连接信息配置 Databricks CLI 和 Databricks Connect。

```groovy
def GITREPO         = "/var/lib/jenkins/workspace/${env.JOB_NAME}"
def GITREPOREMOTE   = "https://github.com/<repo>"
def GITHUBCREDID    = "<github-token>"
def CURRENTRELEASE  = "<release>"
def DBTOKEN         = "<databricks-token>"
def DBURL           = "https://<databricks-instance>"
def SCRIPTPATH      = "${GITREPO}/Automation/Deployments"
def NOTEBOOKPATH    = "${GITREPO}/Workspace"
def LIBRARYPATH     = "${GITREPO}/Libraries"
def BUILDPATH       = "${GITREPO}/Builds/${env.JOB_NAME}-${env.BUILD_NUMBER}"
def OUTFILEPATH     = "${BUILDPATH}/Validation/Output"
def TESTRESULTPATH  = "${BUILDPATH}/Validation/reports/junit"
def WORKSPACEPATH   = "/Shared/<path>"
def DBFSPATH        = "dbfs:<dbfs-path>"
def CLUSTERID       = "<cluster-id>"
def CONDAPATH       = "<conda-path>"
def CONDAENV        = "<conda-env>"

stage('Setup') {
  withCredentials([string(credentialsId: DBTOKEN, variable: 'TOKEN')]) {
  sh """#!/bin/bash
      # Configure Conda environment for deployment & testing
      source ${CONDAPATH}/bin/activate ${CONDAENV}

      # Configure Databricks CLI for deployment
      echo "${DBURL}
      $TOKEN" | databricks configure --token

      # Configure Databricks Connect for testing
      echo "${DBURL}
      $TOKEN
      ${CLUSTERID}
      0
      15001" | databricks-connect configure
     """
  }
}
```

## <a name="get-the-latest-changes"></a>获取最新更改

`Checkout` 阶段会使用 Jenkins 插件将指定分支中的代码下载到“代理执行”代理：

```groovy
stage('Checkout') { // for display purposes
  echo "Pulling ${CURRENTRELEASE} Branch from Github"
  git branch: CURRENTRELEASE, credentialsId: GITHUBCREDID, url: GITREPOREMOTE
}
```

## <a name="develop-unit-tests"></a>开发单元测试

决定如何对代码进行单元测试时，有几个不同的选项可供选择。 对于在 Azure Databricks 笔记本外开发的库代码，此过程类似于传统的软件开发做法。 使用测试框架（如 Python [pytest](https://docs.pytest.org/en/latest/contents.html) 模块）编写单元测试，并使用 JUnit 格式的 XML 文件来存储测试结果。

Azure Databricks 过程的不同之处在于，要测试的代码是 Apache Spark 代码，该代码应在通常运行于本地的 Spark 群集上执行，或在 Azure Databricks 上执行（适用于本例）。  为了满足此要求，请使用 Databricks Connect。 由于此前已配置了该 SDK，因此不需对测试代码进行任何更改即可在 Azure Databricks 群集上执行测试。 你已在 Conda 虚拟环境中安装了 Databricks Connect。 激活 Conda 环境后，将使用 Python 工具 `pytest` 执行测试。你可以为该工具提供测试和生成的输出文件的位置。

### <a name="test-library-code-using-databricks-connect"></a>使用 Databricks Connect 测试库代码

```groovy
stage('Run Unit Tests') {
  try {
      sh """#!/bin/bash
         # Enable Conda environment for tests
         source ${CONDAPATH}/bin/activate ${CONDAENV}

         # Python tests for libs
         python3 -m pytest --junit-xml=${TESTRESULTPATH}/TEST-libout.xml ${LIBRARYPATH}/python/dbxdemo/test*.py || true
         """
  } catch(err) {
    step([$class: 'JUnitResultArchiver', testResults: '--junit-xml=${TESTRESULTPATH}/TEST-*.xml'])
    if (currentBuild.result == 'UNSTABLE')
      currentBuild.result = 'FAILURE'
    throw err
  }
}
```

以下代码片段是可能安装在 Azure Databricks 群集上的库函数。 它是一个简单函数，可将由文本填充的新列添加到 Apache Spark 数据帧中。

```python
# addcol.py
import pyspark.sql.functions as F

def with_status(df):
    return df.withColumn("status", F.lit("checked"))
```

此测试将一个模拟数据帧对象传递到 `with_status` 函数，该函数在 `addcol.py` 中定义。  然后，结果将与一个包含预期值的数据帧对象进行比较。 如果值匹配（在此示例中会如此），则测试通过。

```python
# test-addcol.py
import pytest

from dbxdemo.spark import get_spark
from dbxdemo.appendcol import with_status

class TestAppendCol(object):

  def test_with_status(self):
    source_data = [
        ("paula", "white", "paula.white@example.com"),
        ("john", "baer", "john.baer@example.com")
    ]
    source_df = get_spark().createDataFrame(
        source_data,
        ["first_name", "last_name", "email"]
    )

    actual_df = with_status(source_df)

    expected_data = [
        ("paula", "white", "paula.white@example.com", "checked"),
        ("john", "baer", "john.baer@example.com", "checked")
    ]
    expected_df = get_spark().createDataFrame(
        expected_data,
        ["first_name", "last_name", "email", "status"]
    )

    assert(expected_df.collect() == actual_df.collect())
```

## <a name="package-library-code"></a>打包库代码

在 `Package` 阶段，你会将库代码打包到一个 Python wheel 中。

```groovy
stage('Package') {
  sh """#!/bin/bash
      # Enable Conda environment for tests
      source ${CONDAPATH}/bin/activate ${CONDAENV}

      # Package Python library to wheel
      cd ${LIBRARYPATH}/python/dbxdemo
      python3 setup.py sdist bdist_wheel
     """
}
```

## <a name="generate-and-store-a-deployment-artifact"></a>生成并存储部署项目

为 Azure Databricks 生成部署项目涉及收集所有新的或更新的代码并将其部署到相应的 Azure Databricks 环境。 在 `Build Artifact` 阶段，请添加要部署到工作区的笔记本代码、由生成过程生成的任何 `whl` 库，以及用于存档的测试结果摘要。 为此，请使用 `git diff` 来标记最近的 git 合并中已包含的所有新文件。 这只是一个示例方法，因此管道中的实现可能会有所不同，但目标是添加所有适用于当前版本的文件。

```groovy
stage('Build Artifact') {
  sh """mkdir -p ${BUILDPATH}/Workspace
        mkdir -p ${BUILDPATH}/Libraries/python
        mkdir -p ${BUILDPATH}/Validation/Output
        #Get Modified Files
        git diff --name-only --diff-filter=AMR HEAD^1 HEAD | xargs -I '{}' cp --parents -r '{}' ${BUILDPATH}

        # Get packaged libs
        find ${LIBRARYPATH} -name '*.whl' | xargs -I '{}' cp '{}' ${BUILDPATH}/Libraries/python/

        # Generate artifact
        tar -czvf Builds/latest_build.tar.gz ${BUILDPATH}
     """
  archiveArtifacts artifacts: 'Builds/latest_build.tar.gz'
}
```

## <a name="deploy-artifacts"></a>部署项目

在 `Deploy` 阶段，你将使用 [Databricks CLI](../cli/index.md)，它像之前使用的 Databricks Connect 模块一样安装在 Conda 环境中，因此你必须为此 shell 会话激活它。
使用工作区 CLI 和 DBFS CLI 分别上传笔记本和库：

```bash
databricks workspace import_dir <local build path> <remote workspace path>
dbfs cp -r <local build path> <remote dbfs path>
```

```groovy
stage('Deploy') {
  sh """#!/bin/bash
        # Enable Conda environment for tests
        source ${CONDAPATH}/bin/activate ${CONDAENV}

        # Use Databricks CLI to deploy notebooks
        databricks workspace import_dir ${BUILDPATH}/Workspace ${WORKSPACEPATH}

        dbfs cp -r ${BUILDPATH}/Libraries/python ${DBFSPATH}
     """
  withCredentials([string(credentialsId: DBTOKEN, variable: 'TOKEN')]) {
    sh """#!/bin/bash

        #Get space delimited list of libraries
        LIBS=\$(find ${BUILDPATH}/Libraries/python/ -name '*.whl' | sed 's#.*/##' | paste -sd " ")

        #Script to uninstall, reboot if needed & instsall library
        python3 ${SCRIPTPATH}/installWhlLibrary.py --workspace=${DBURL}\
                  --token=$TOKEN\
                  --clusterid=${CLUSTERID}\
                  --libs=\$LIBS\
                  --dbfspath=${DBFSPATH}
       """
  }
}
```

若要在 Azure Databricks 群集上安装新版本的库，必须先卸载现有库。
为此，请在 Python 脚本中调用 [Databricks REST API](../api/latest/index.md) 以执行以下步骤：

1. 检查是否已安装该库。
2. 卸载该库。
3. 如果执行了任何卸载，请重启群集。
   1. 请等到群集再次运行，然后继续操作。
4. 安装该库。

```python
# installWhlLibrary.py
#!/usr/bin/python3
import json
import requests
import sys
import getopt
import time

def main():
  workspace = ''
  token = ''
  clusterid = ''
  libs = ''
  dbfspath = ''

  try:
      opts, args = getopt.getopt(sys.argv[1:], 'hstcld',
                                 ['workspace=', 'token=', 'clusterid=', 'libs=', 'dbfspath='])
  except getopt.GetoptError:
      print(
          'installWhlLibrary.py -s <workspace> -t <token> -c <clusterid> -l <libs> -d <dbfspath>')
      sys.exit(2)

  for opt, arg in opts:
      if opt == '-h':
          print(
              'installWhlLibrary.py -s <workspace> -t <token> -c <clusterid> -l <libs> -d <dbfspath>')
          sys.exit()
      elif opt in ('-s', '--workspace'):
          workspace = arg
      elif opt in ('-t', '--token'):
          token = arg
      elif opt in ('-c', '--clusterid'):
          clusterid = arg
      elif opt in ('-l', '--libs'):
          libs=arg
      elif opt in ('-d', '--dbfspath'):
          dbfspath=arg

  print('-s is ' + workspace)
  print('-t is ' + token)
  print('-c is ' + clusterid)
  print('-l is ' + libs)
  print('-d is ' + dbfspath)

  libslist = libs.split()

  # Uninstall Library if exists on cluster
  i=0
  for lib in libslist:
      dbfslib = dbfspath + lib
      print(dbfslib + ' before:' + getLibStatus(workspace, token, clusterid, dbfslib))

      if (getLibStatus(workspace, token, clusterid, dbfslib) != 'not found'):
          print(dbfslib + " exists. Uninstalling.")
          i = i + 1
          values = {'cluster_id': clusterid, 'libraries': [{'whl': dbfslib}]}

          resp = requests.post(workspace + '/api/2.0/libraries/uninstall', data=json.dumps(values), auth=("token", token))
          runjson = resp.text
          d = json.loads(runjson)
          print(dbfslib + ' after:' + getLibStatus(workspace, token, clusterid, dbfslib))

  # Restart if libraries uninstalled
  if i > 0:
      values = {'cluster_id': clusterid}
      print("Restarting cluster:" + clusterid)
      resp = requests.post(workspace + '/api/2.0/clusters/restart', data=json.dumps(values), auth=("token", token))
      restartjson = resp.text
      print(restartjson)

      p = 0
      waiting = True
      while waiting:
          time.sleep(30)
          clusterresp = requests.get(workspace + '/api/2.0/clusters/get?cluster_id=' + clusterid,
                                 auth=("token", token))
          clusterjson = clusterresp.text
          jsonout = json.loads(clusterjson)
          current_state = jsonout['state']
          print(clusterid + " state:" + current_state)
          if current_state in ['RUNNING','INTERNAL_ERROR', 'SKIPPED'] or p >= 10:
              break
          p = p + 1

  # Install Libraries
  for lib in libslist:
      dbfslib = dbfspath + lib
      print("Installing " + dbfslib)
      values = {'cluster_id': clusterid, 'libraries': [{'whl': dbfslib}]}

      resp = requests.post(workspace + '/api/2.0/libraries/install', data=json.dumps(values), auth=("token", token))
      runjson = resp.text
      d = json.loads(runjson)
      print(dbfslib + ' after:' + getLibStatus(workspace, token, clusterid, dbfslib))

def getLibStatus(workspace, token, clusterid, dbfslib):
  resp = requests.get(workspace + '/api/2.0/libraries/cluster-status?cluster_id='+ clusterid, auth=("token", token))
  libjson = resp.text
  d = json.loads(libjson)
  if (d.get('library_statuses')):
      statuses = d['library_statuses']

      for status in statuses:
          if (status['library'].get('whl')):
              if (status['library']['whl'] == dbfslib):
                  return status['status']
              else:
                  return "not found"
  else:
      # No libraries found
      return "not found"

if __name__ == '__main__':
  main()
```

### <a name="test-notebook-code-using-another-notebook"></a>使用另一笔记本测试笔记本代码

部署项目后，请务必运行集成测试，以确保所有代码在新环境中协同工作。 为此，可以运行包含断言的笔记本来测试部署。 在这种情况下，你使用的是在单元测试中使用的同一测试，但现在它将从刚安装在群集上的 `whl` 中导入已安装的 `appendcol` 库。

若要自动执行此测试，并将其包括在 CI/CD 管道中，请使用 Databricks REST API 从 Jenkins 服务器运行笔记本。 因此，你可以使用 `pytest` 来检查笔记本运行是成功还是失败。  如果笔记本中的断言失败，则会在 REST API 返回的 JSON 输出中进行相应显示，并随后在 JUnit 测试结果中显示。

```groovy
stage('Run Integration Tests') {
  withCredentials([string(credentialsId: DBTOKEN, variable: 'TOKEN')]) {
      sh """python3 ${SCRIPTPATH}/executenotebook.py --workspace=${DBURL}\
                      --token=$TOKEN\
                      --clusterid=${CLUSTERID}\
                      --localpath=${NOTEBOOKPATH}/VALIDATION\
                      --workspacepath=${WORKSPACEPATH}/VALIDATION\
                      --outfilepath=${OUTFILEPATH}
         """
  }
  sh """sed -i -e 's #ENV# ${OUTFILEPATH} g' ${SCRIPTPATH}/evaluatenotebookruns.py
        python3 -m pytest --junit-xml=${TESTRESULTPATH}/TEST-notebookout.xml ${SCRIPTPATH}/evaluatenotebookruns.py || true
     """
}
```

此阶段调用两个 Python 自动化脚本。 第一个脚本 `executenotebook.py` 使用用于提交匿名作业的[作业运行提交](../api/latest/jobs.md#runs-submit)终结点来运行此笔记本。 由于此终结点是异步的，因此它使用 REST 调用最初返回的作业 ID 来轮询作业的状态。 作业完成后，JSON 输出会保存到在调用时传递的函数参数指定的路径。

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
  workspace = ''
  token = ''
  clusterid = ''
  localpath = ''
  workspacepath = ''
  outfilepath = ''

  try:
      opts, args = getopt.getopt(sys.argv[1:], 'hs:t:c:lwo',
                                 ['workspace=', 'token=', 'clusterid=', 'localpath=', 'workspacepath=', 'outfilepath='])
  except getopt.GetoptError:
      print(
          'executenotebook.py -s <workspace> -t <token>  -c <clusterid> -l <localpath> -w <workspacepath> -o <outfilepath>)')
      sys.exit(2)

  for opt, arg in opts:
      if opt == '-h':
          print(
              'executenotebook.py -s <workspace> -t <token> -c <clusterid> -l <localpath> -w <workspacepath> -o <outfilepath>')
          sys.exit()
      elif opt in ('-s', '--workspace'):
          workspace = arg
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

  print('-s is ' + workspace)
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

  # run each element in list
  for notebook in notebooks:
      nameonly = os.path.basename(notebook[0])
      workspacepath = notebook[1]

      name, file_extension = os.path.splitext(nameonly)

      # workpath removes extension
      fullworkspacepath = workspacepath + '/' + name

      print('Running job for:' + fullworkspacepath)
      values = {'run_name': name, 'existing_cluster_id': clusterid, 'timeout_seconds': 3600, 'notebook_task': {'notebook_path': fullworkspacepath}}

      resp = requests.post(workspace + '/api/2.0/jobs/runs/submit',
                           data=json.dumps(values), auth=("token", token))
      runjson = resp.text
      print("runjson:" + runjson)
      d = json.loads(runjson)
      runid = d['run_id']

      i=0
      waiting = True
      while waiting:
          time.sleep(10)
          jobresp = requests.get(workspace + '/api/2.0/jobs/runs/get?run_id='+str(runid),
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

第二个脚本 `evaluatenotebookruns.py` 定义 `test_job_run` 函数，该函数分析并评估 JSON，以确定笔记本中的断言语句是成功还是失败。
另一测试 `test_performance` 会捕获运行时间长于预期时间的测试。

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

如前面的单元测试阶段所示，可以使用 `pytest` 来运行测试并生成结果摘要。

### <a name="publish-test-results"></a>发布测试结果

JSON 结果会存档，并且测试结果会通过 `junit` Jenkins 插件发布到 Jenkins。 这样，你便可以将与生成过程的状态相关的报表和仪表板可视化。

```groovy
stage('Report Test Results') {
  sh """find ${OUTFILEPATH} -name '*.json' -exec gzip --verbose {} \\;
        touch ${TESTRESULTPATH}/TEST-*.xml
     """
  junit "**/reports/junit/*.xml"
}
```

> [!div class="mx-imgBorder"]
> ![Jenkins 测试结果](../../_static/images/jenkins-test.png)

此时，CI/CD 管道已完成集成和部署周期。 通过自动执行此过程，可以确保代码已经通过有效、一致且可重复的过程进行了测试和部署。