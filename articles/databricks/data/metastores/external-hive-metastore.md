---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: 外部 Apache Hive 元存储 - Azure Databricks
description: 了解如何连接到 Azure Databricks 中的外部 Apache Hive 元存储。
ms.openlocfilehash: d70cfd00d993074fcf1d1f45971846c0417eaa96
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121836"
---
# <a name="external-apache-hive-metastore"></a><a id="external-apache-hive-metastore"> </a><a id="external-hive-metastore"> </a>外部 Apache Hive 元存储

本文介绍了如何设置 Azure Databricks 群集以连接到现有的外部 Apache Hive 元存储。 其中提供了有关建议的元存储设置和群集配置要求的信息，以及有关配置群集以连接到外部元存储的说明。 下表汇总了 Databricks Runtime 的每个版本支持的 Hive 元存储版本。

| Databricks 运行时版本     | 0.13 - 1.2.1          | 2.1          | 2.2          | 2.3          | 3.1.0          |
|--------------------------------|-----------------------|--------------|--------------|--------------|----------------|
| 7.x                            | 是                   | 是          | 是          | 是          | 是            |
| 6.x                            | 是                   | 是          | 是          | 是          | 是            |
| 5.3 及更高版本                  | 是                   | 是          | 是          | 是          | 是            |
| 5.1 - 5.2 和 4.x              | 是                   | 是          | 是          | 是          | 否             |
| 3.x                            | 是                   | 是          | 否           | 否           | 否             |

> [!IMPORTANT]
>
> * SQL Server 不能用作 Hive 2.0 及更高版本的基础元存储数据库；但是，Azure SQL 数据库可以，并且在本文中从头到尾用作示例。
> * 可以使用 HDInsight 群集的 Hive 1.2.0 或 1.2.1 元存储作为外部元存储。 请参阅[使用 Azure HDInsight 中的外部元数据存储](/hdinsight/hdinsight-use-external-metadata-stores)。
> * 如果使用 Azure Database for MySQL 作为外部元存储，则必须在服务器端数据库配置中将 `lower_case_table_names` 属性的值从 1（默认值）更改为 2。 有关详细信息，请参阅[标识符区分大小写](https://dev.mysql.com/doc/refman/5.6/en/identifier-case-sensitivity.html)。

## <a name="hive-metastore-setup"></a>Hive 元存储设置

在群集内运行的元存储客户端直接使用 JDBC 连接到基础元存储数据库。

若要测试从群集到元存储的网络连接，可以在笔记本中运行以下命令：

```bash
%sh
nc -vz <DNS name> <port>
```

where

* `<DNS name>` 是 Azure SQL 数据库的服务器名称。
* `<port>` 是数据库的端口。

## <a name="cluster-configurations"></a>群集配置

你必须设置两组配置选项才能将群集连接到外部元存储：

* [Spark 选项](#spark-options)为 Spark 配置元存储客户端的 Hive 元存储版本和 JAR。
* [Hive 选项](#hive-options)配置元存储客户端以连接到外部元存储。

### <a name="spark-configuration-options"></a><a id="spark-configuration-options"> </a><a id="spark-options"> </a>Spark 配置选项

将 `spark.sql.hive.metastore.version` 设置为你的 Hive 元存储的版本，并将 `spark.sql.hive.metastore.jars` 设置如下：

* Hive 0.13：不设置 `spark.sql.hive.metastore.jars`。
* Hive 1.2.0 或 1.2.1（低于 7.0 的 Databricks Runtime 版本）：将 `spark.sql.hive.metastore.jars` 设置为 `builtin`。

  > [!NOTE]
  >
  > 在 Databricks Runtime 7.0 及更高版本上，Hive 1.2.0 和 1.2.1 不是内置的元存储。 如果要将 Hive 1.2.0 或 1.2.1 与 Databricks Runtime 7.0 及更高版本一起使用，请按照[下载元存储 jar 并指向它们](#download-the-metastore-jars-and-point-to-them)中所述的过程进行操作。

* Hive 2.3（Databricks Runtime 7.0 及更高版本）：将 `spark.sql.hive.metastore.jars` 设置为 `builtin`。
* 对于所有其他 Hive 版本，Azure Databricks 建议你下载元存储 JAR 并将配置 `spark.sql.hive.metastore.jars` 设置为指向下载的 JAR，方法是使用[下载元存储 jar 并指向它们](#download-the-metastore-jars-and-point-to-them)中所述的过程。

#### <a name="download-the-metastore-jars-and-point-to-them"></a>下载元存储 jar 并指向它们

1. 创建一个群集，将 `spark.sql.hive.metastore.jars` 设置为 `maven`，将 `spark.sql.hive.metastore.version` 设置为与你的元存储的版本匹配。
2. 在群集运行时搜索驱动程序日志，找到如下所示的行：

   ```
   17/11/18 22:41:19 INFO IsolatedClientLoader: Downloaded metastore jars to <path>
   ```

   目录 `<path>` 是已下载的 JAR 在群集的驱动程序节点中的位置。

   另外，你可以在 Scala 笔记本中运行以下代码来输出 JAR 的位置：

   ```scala
   import com.typesafe.config.ConfigFactory
   val path = ConfigFactory.load().getString("java.io.tmpdir")

   println(s"\nHive JARs are downloaded to the path: $path \n")
   ```

3. 运行 `%sh cp -r <path> /dbfs/hive_metastore_jar`（将 `<path>` 替换为你的群集的信息），以通过驱动程序节点中的 FUSE 客户端将此目录复制到 DBFS 中名为 `hive_metastore_jar` 的目录。
4. 创建一个会将 `/dbfs/hive_metastore_jar` 复制到节点的本地文件系统中的[初始化脚本](../../clusters/init-scripts.md)，确保初始化脚本在访问 DBFS FUSE 客户端之前进入睡眠状态几秒钟。 这可确保客户端准备就绪。
5. 设置 `spark.sql.hive.metastore.jars` 以使用此目录。 如果初始化脚本将 `/dbfs/hive_metastore_jar` 复制到 `/databricks/hive_metastore_jars/`，请将 `spark.sql.hive.metastore.jars` 设置为 `/databricks/hive_metastore_jars/*`。 该位置必须包含尾随 `/*`。
6. 重启群集。

### <a name="hive-configuration-options"></a><a id="hive-configuration-options"> </a><a id="hive-options"> </a>Hive 配置选项

本部分介绍了特定于 Hive 的选项。

若要使用本地模式连接到外部元存储，请设置以下 Hive 配置选项：

```ini
# JDBC connect string for a JDBC metastore
javax.jdo.option.ConnectionURL <mssql-connection-string>

# Username to use against metastore database
javax.jdo.option.ConnectionUserName <mssql-username>

# Password to use against metastore database
javax.jdo.option.ConnectionPassword <mssql-password>

# Driver class name for a JDBC metastore
javax.jdo.option.ConnectionDriverName com.microsoft.sqlserver.jdbc.SQLServerDriver
```

where

* `<mssql-connection-string>` 是 JDBC 连接字符串（可以在 Azure 门户中获取）。 不需要在连接字符串中包含用户名和密码，因为它们将由 `javax.jdo.option.ConnectionUserName` 和 `javax.jdo.option.ConnectionDriverName` 设置。
* `<mssql-username>` 和 `<mssql-password>` 指定具有数据库读/写访问权限的 Azure SQL 数据库帐户的用户名和密码。

> [!NOTE]
>
> 对于生产环境，建议将 `hive.metastore.schema.verification` 设置为 `true`。 这可防止当元存储客户端版本与元存储数据库版本不匹配时，Hive 元存储客户端隐式修改元存储数据库架构。 为低于 Hive 1.2.0 的元存储客户端版本启用此设置时，请确保元存储客户端对元存储数据库具有写入权限（以防止 [HIVE-9749](https://issues.apache.org/jira/browse/HIVE-9749) 中所述的问题）。
>
> * 对于 Hive 元存储 1.2.0 及更高版本，请将 `hive.metastore.schema.verification.record.version` 设置为 `true` 以启用 `hive.metastore.schema.verification`。
> * 对于 Hive 元存储 2.1.1 及更高版本，请将 `hive.metastore.schema.verification.record.version` 设置为 `true` ，因为它默认设置为 `false`。

## <a name="set-up-an-external-metastore-using-the-ui"></a>使用 UI 设置外部元存储

若要使用 Azure Databricks UI 设置外部元存储，请执行以下操作：

1. 单击边栏中的“群集”按钮。
2. 单击“创建群集”。****
3. 输入以下 [Spark 配置选项](../../clusters/configure.md#spark-config)：

   ```ini
   # Hive-specific configuration options.
   # spark.hadoop prefix is added to make sure these Hive specific options propagate to the metastore client.
   # JDBC connect string for a JDBC metastore
   spark.hadoop.javax.jdo.option.ConnectionURL <mssql-connection-string>

   # Username to use against metastore database
   spark.hadoop.javax.jdo.option.ConnectionUserName <mssql-username>

   # Password to use against metastore database
   spark.hadoop.javax.jdo.option.ConnectionPassword <mssql-password>

   # Driver class name for a JDBC metastore
   spark.hadoop.javax.jdo.option.ConnectionDriverName com.microsoft.sqlserver.jdbc.SQLServerDriver

   # Spark specific configuration options
   spark.sql.hive.metastore.version <hive-version>
   # Skip this one if <hive-version> is 0.13.x.
   spark.sql.hive.metastore.jars <hive-jar-source>
   ```

4. 根据[配置群集](../../clusters/configure.md#cluster-configurations)中的说明继续你的群集配置。
5. 单击“创建群集”以创建群集。

## <a name="set-up-an-external-metastore-using-an-init-script"></a>使用初始化脚本设置外部元存储

[初始化脚本](../../clusters/init-scripts.md)允许你连接到现有 Hive 元存储，无需手动设置所需的配置。

1. 创建要在其中存储初始化脚本的基目录（如果不存在）。 下面的示例使用 `dbfs:/databricks/scripts`。
2. 在笔记本中运行以下代码片段。 此代码片段在 [Databricks 文件系统 (DBFS)](../databricks-file-system.md) 中创建初始化脚本 `/databricks/scripts/external-metastore.sh`。 此外，你可以使用 [DBFS REST API 的 put 操作](../../dev-tools/api/latest/dbfs.md#dbfsdbfsserviceput)创建初始化脚本。 每当名称指定为 `<cluster-name>` 的群集启动时，此初始化脚本都会将所需的配置选项写入到一个名为 `00-custom-spark.conf` 的配置文件中，该文件采用类似于 JSON 的格式，位于群集的每个节点中的 `/databricks/driver/conf/` 下。 Azure Databricks 在 `/databricks/driver/conf/spark-branch.conf` 文件中提供了默认的 Spark 配置。 `/databricks/driver/conf` 目录中的配置文件以反向字母顺序应用。 如果要更改 `00-custom-spark.conf` 文件的名称，请确保它在 `spark-branch.conf` 文件之前持续应用。

   ```scala
   dbutils.fs.put(
       "/databricks/scripts/external-metastore.sh",
       """#!/bin/sh
         |# Loads environment variables to determine the correct JDBC driver to use.
         |source /etc/environment
         |# Quoting the label (i.e. EOF) with single quotes to disable variable interpolation.
         |cat << 'EOF' > /databricks/driver/conf/00-custom-spark.conf
         |[driver] {
         |    # Hive specific configuration options.
         |    # spark.hadoop prefix is added to make sure these Hive specific options will propagate to the metastore client.
         |    # JDBC connect string for a JDBC metastore
         |    "spark.hadoop.javax.jdo.option.ConnectionURL" = "<mssql-connection-string>"
         |
         |    # Username to use against metastore database
         |    "spark.hadoop.javax.jdo.option.ConnectionUserName" = "<mssql-username>"
         |
         |    # Password to use against metastore database
         |    "spark.hadoop.javax.jdo.option.ConnectionPassword" = "<mssql-password>"
         |
         |    # Driver class name for a JDBC metastore
         |    "spark.hadoop.javax.jdo.option.ConnectionDriverName" = "com.microsoft.sqlserver.jdbc.SQLServerDriver"
         |
         |    # Spark specific configuration options
         |    "spark.sql.hive.metastore.version" = "<hive-version>"
         |    # Skip this one if <hive-version> is 0.13.x.
         |    "spark.sql.hive.metastore.jars" = "<hive-jar-source>"
         |}
         |EOF
         |""".stripMargin,
       overwrite = true
   )
   ```

3. 使用初始化脚本配置你的群集。
4. 重启群集。

## <a name="troubleshooting"></a>故障排除

**群集未启动（由于初始化脚本设置不正确）**

如果用于设置外部元存储的初始化脚本导致群集创建失败，请将初始化脚本配置为[日志](../../clusters/init-scripts.md#init-script-log)，然后使用日志调试初始化脚本。

**SQL 语句出错：InvocationTargetException**

* 完整异常堆栈跟踪中的错误消息模式：

  ```
  Caused by: javax.jdo.JDOFatalDataStoreException: Unable to open a test connection to the given database. JDBC url = [...]
  ```

  外部元存储 JDBC 连接信息配置不正确。 请验证所配置的主机名、端口、用户名、密码和 JDBC 驱动程序类名称。 此外，请确保用户名具有访问元存储数据库的适当权限。

* 完整异常堆栈跟踪中的错误消息模式：

  ```
  Required table missing : "`DBS`" in Catalog "" Schema "". DataNucleus requires this table to perform its persistence operations. [...]
  ```

  外部元存储数据库未正确初始化。 验证你是否已创建元存储数据库并将正确的数据库名称放在 JDBC 连接字符串中。 然后，使用以下两个 Spark 配置选项启动新群集：

  ```ini
  datanucleus.autoCreateSchema true
  datanucleus.fixedDatastore false
  ```

  这样，当 Hive 客户端库尝试访问表但找不到它们时，它会尝试在元存储数据库中自动创建并初始化表。

**SQL 语句出错：AnalysisException：无法实例化 org.apache.hadoop.hive.metastore.HiveMetastoreClient**

完整异常堆栈跟踪中的错误消息：

```
The specified datastore driver (driver name) was not found in the CLASSPATH
```

群集配置为使用不正确的 JDBC 驱动程序。

**将 datanucleus.autoCreateSchema 设置为 true 的操作不按预期工作**

默认情况下，Databricks 还会将 `datanucleus.fixedDatastore` 设置为 `true`，这会阻止意外更改元存储数据库的结构。 因此，即使将 `datanucleus.autoCreateSchema` 设置为 `true`，Hive 客户端库也无法创建元存储表。 通常，对于生产环境而言，此策略会更安全，因为它可以防止元存储数据库意外升级。

如果确实需要使用 `datanucleus.autoCreateSchema` 来帮助初始化元存储数据库，请确保将 `datanucleus.fixedDatastore` 设置为 `false`。 此外，你可能想要在初始化元存储数据库后翻转这两个标志，以便为你的生产环境提供更好的保护。