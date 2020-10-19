---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: 库 CLI - Azure Databricks
description: 了解如何使用 Databricks 库命令行接口。
ms.openlocfilehash: 9802b44f3e8744724c4cd1fdc87cc7517050f455
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937782"
---
# <a name="libraries-cli"></a>库 CLI

可以通过将 Databricks 库 CLI 子命令追加到 `databricks libraries` 之后来运行这些命令。

```bash
databricks libraries -h
```

```
Usage: databricks libraries [OPTIONS] COMMAND [ARGS]...

  Utility to interact with libraries.

Options:
  -v, --version  [VERSION]
  -h, --help     Show this message and exit.

Commands:
  all-cluster-statuses  Get the status of all libraries.
  cluster-status        Get the status of all libraries for a cluster.
    Options:
      --cluster-id CLUSTER_ID   Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#/setting/clusters/$CLUSTER_ID/configuration.
  install               Install a library on a cluster.
    Options:
      --cluster-id CLUSTER_ID   Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#/setting/clusters/$CLUSTER_ID/configuration.
      --jar TEXT                JAR on DBFS or WASB.
      --egg TEXT                Egg on DBFS or WASB.
      --whl TEXT                Wheel or zipped wheelhouse on DBFS or WASB. Supported in CLI 0.8.2 and above.
      --maven-coordinates TEXT  Maven coordinates in the form of GroupId:ArtifactId:Version (i.e.org.jsoup:jsoup:1.7.2).
      --maven-repo TEXT         Maven repository to install the Maven package from. If omitted, both Maven Repository and Spark Packages are searched.
      --maven-exclusion TEXT    List of dependences to exclude. For example: --maven-exclusion "slf4j:slf4j" --maven-exclusion "*:hadoop-client".
      --pypi-package TEXT       The name of the PyPI package to install. An optional exact version specification is also supported. Examples "simplejson" and "simplejson==3.8.0".
      --pypi-repo TEXT          The repository where the package can be found. If not specified, the default pip index is used.
      --cran-package TEXT       The name of the CRAN package to install.
      --cran-repo TEXT          The repository where the package can be found. If not specified, the default CRAN repo is used.
  list                  Shortcut to `all-cluster-statuses` or `cluster-status`.
    Options:
      --cluster-id CLUSTER_ID   Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#/setting/clusters/$CLUSTER_ID/configuration.
  uninstall             Uninstall a library on a cluster.
    Options:
      --cluster-id CLUSTER_ID   Can be found in the URL at https://<databricks-instance>/?o=<16-digit-number>#/setting/clusters/$CLUSTER_ID/configuration. [required]
      --all                     Uninstall all libraries.
      --jar TEXT                JAR on DBFS or WASB.
      --egg TEXT                Egg on DBFS or WASB.
      --whl TEXT                Wheel or zipped wheelhouse on DBFS or WASB. Supported in CLI 0.8.2 and above.
      --maven-coordinates TEXT  Maven coordinates in the form of GroupId:ArtifactId:Version (i.e.org.jsoup:jsoup:1.7.2).
      --maven-repo TEXT         Maven repository to install the Maven package from. If omitted, both Maven Repository and Spark Packages are searched.
      --maven-exclusion TEXT    List of dependences to exclude. For example: --maven-exclusion "slf4j:slf4j" --maven-exclusion "*:hadoop-client".
      --pypi-package TEXT       The name of the PyPI package to install. An optional exact version specification is also supported. Examples "simplejson" and "simplejson==3.8.0".
      --pypi-repo TEXT          The repository where the package can be found. If not specified, the default pip index is used.
      --cran-package TEXT       The name of the CRAN package to install.
      --cran-repo TEXT          The repository where the package can be found. If not specified, the default CRAN repo is used.
```

## <a name="install-a-jar-from-dbfs"></a>从 DBFS 安装 JAR

```bash
databricks libraries install --cluster-id $CLUSTER_ID --jar dbfs:/test-dir/test.jar
```

## <a name="list-library-statuses-for-a-cluster"></a>列出群集的库状态

```bash
databricks libraries list --cluster-id $CLUSTER_ID
```