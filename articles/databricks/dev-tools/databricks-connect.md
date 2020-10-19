---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/06/2020
title: Databricks Connect - Azure Databricks
description: 了解如何使用 Databricks Connect 将喜欢的 IDE、笔记本服务器或自定义应用程序连接到 Azure Databricks 群集。
ms.openlocfilehash: 76383312b1c7c14aa05057a9476ff6152ecedf5f
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937726"
---
# <a name="databricks-connect"></a>Databricks Connect

通过 Databricks Connect，可将喜欢的 IDE（IntelliJ、Eclipse、PyCharm、RStudio 和 Visual Studio）、笔记本服务器（Zeppelin 和 Jupyter）和其他自定义应用程序连接到 Azure Databricks 群集，并运行 Apache Spark 代码。

本文将介绍 Databricks Connect 的工作原理，引导你完成 Databricks Connect 的入门步骤，阐述如何解决使用 Databricks Connect 时可能出现的问题，还将介绍使用 Databricks Connect 运行与在 Azure Databricks 笔记本中运行之间的区别。

## <a name="overview"></a>概述

Databricks Connect 是一个适用于 Apache Spark 的客户端库。 借助它，你可使用 Spark 本机 API 编写作业，并让它们在 Azure Databricks 群集上（而不是在本地 Spark 会话中）远程执行。

例如，使用 Databricks Connect 运行 DataFrame 命令 `spark.read.parquet(...).groupBy(...).agg(...).show()` 时，作业的分析和计划任务将在本地计算机上运行。 然后，作业的逻辑表示形式会发送到在 Azure Databricks 中运行的 Spark 服务器，以便在群集中执行。

可使用 Databricks Connect 执行以下操作：

* 从任何 Python、Java、Scala 或 R 应用程序运行大规模的 Spark 作业。 现可在任何能执行 `import pyspark`、`import org.apache.spark` 或 `require(SparkR)` 的位置直接从应用程序运行 Spark 作业，无需安装任何 IDE 插件，也无需使用 Spark 提交脚本。
* 即使使用远程群集，也要在 IDE 中单步执行和调试代码。
* 开发库时快速循环访问。 在 Databricks Connect 中更改 Python 或 Java 库依赖项后，无需重启群集，这是因为群集中的每个客户端会话彼此隔离。
* 关闭空闲群集而不丢失工作。 客户端应用程序与群集是分离的，因此它不受群集重启或升级的影响，这通常会导致你丢失笔记本中定义的所有变量、RDD 和 DataFrame 对象。

## <a name="requirements"></a>要求

> [!NOTE]
>
> 在 Databricks Runtime 7.1 中，Databricks 建议始终使用 Databricks Connect 的最新版本。

* 客户端 Python 安装的次要版本必须与 Azure Databricks 群集的 Python 次要版本（3.5、3.6 或 3.7）相同。 Databricks Runtime 5.5 LTS 具有 Python 3.5，Databricks Runtime 5.5 LTS ML 具有 Python 3.6，Databricks Runtime 6.1（及更高版本）和 Databricks Runtime 6.1 ML（及更高版本）具有 Python 3.7。

  例如，如果在本地开发环境中使用 Conda，并且群集运行的是 Python 3.5，则必须使用该版本创建一个环境，例如：

  ```bash
  conda create --name dbconnect python=3.5
  ```

* Java 8。 客户端不支持 Java 11。

## <a name="set-up-client"></a>设置客户端

### <a name="step-1-install-the-client"></a>步骤 1：安装客户端

1. 卸载 PySpark。

   ```bash
   pip uninstall pyspark
   ```

2. 安装 Databricks Connect 客户端。

   ```bash
   pip install -U databricks-connect==5.5.*  # or 6.*.* or 7.1.* to match your cluster version. 6.1-6.6 and 7.1 are supported
   ```

### <a name="step-2-configure-connection-properties"></a>步骤 2：配置连接属性

1. 收集以下配置属性：
   * **URL**：[工作区 URL](../workspace/workspace-details.md#workspace-url)。
   * **用户令牌：** [个人访问令牌](api/latest/authentication.md#token-management)。
   * **群集 ID：** 你创建的群集的 ID。 可从 URL 获取群集 ID。 此处的群集 ID 是 `1108-201635-xxxxxxxx`。

     > [!div class="mx-imgBorder"]
     > ![群集 ID](../_static/images/db-connect/cluster-id-azure.png)

   * **组织 ID** 每个工作区都具有唯一的组织 ID。 请参阅[获取工作区、群集、笔记本、模型和作业标识符](../workspace/workspace-details.md)。
   * **端口**：Databricks Connect 连接到的端口。 默认端口为 `15001`。 如果将群集配置为使用其他端口，例如在之前的 Azure Databricks 说明中提供的 `8787`，则使用所配置的端口号。
2. 配置连接。 可使用 CLI、SQL 配置或环境变量。 配置方法的优先级从高到低为：SQL 配置密钥、CLI 和环境变量。
   * CLI
     1. 运行 `databricks-connect`。

        ```bash
        databricks-connect configure
        ```

        许可证显示：

        ```
        Copyright (2018) Databricks, Inc.

        This library (the "Software") may not be used except in connection with the
        Licensee's use of the Databricks Platform Services pursuant to an Agreement
          ...
        ```

     1. 接受许可证并提供配置值。

        ```
        Do you accept the above agreement? [y/N] y
        Set new config values (leave input empty to accept default):
        Databricks Host [no current value, must start with https://]: <databricks-url>
        Databricks Token [no current value]: <databricks-token>
        Cluster ID (e.g., 0921-001415-jelly628) [no current value]: <cluster-id>
        Org ID (Azure-only, see ?o=orgId in URL) [0]: <org-id>
        Port [15001]: <port>
        ```

   * SQL 配置或环境变量。 设置 SQL 配置密钥（例如 `sql("set config=value")`）和环境变量，如下所示：

     | 参数            | SQL 配置密钥                         | 环境变量名称                       |
     |----------------------|----------------------------------------|-------------------------------------------------|
     | Databricks 主机      | spark.databricks.service.address       | DATABRICKS_ADDRESS                              |
     | Databricks 令牌     | spark.databricks.service.token         | DATABRICKS_API_TOKEN                            |
     | 群集 ID           | spark.databricks.service.clusterId     | DATABRICKS_CLUSTER_ID                           |
     | 组织 ID               | spark.databricks.service.orgId         | DATABRICKS_ORG_ID                               |
     | 端口                 | spark.databricks.service.port          | DATABRICKS_PORT（仅限版本高于 5.4 的 Databricks Runtime） |

     > [!IMPORTANT]
     >
     > 建议不要在 SQL 配置中放置令牌。

3. 测试与 Azure Databricks 之间的连接。

   ```bash
   databricks-connect test
   ```

   如果配置的群集未运行，测试将启动群集，该群集将保持运行，直到其配置的自动终止时间为止。 输出应类似于以下内容：

   ```
   * PySpark is installed at /.../3.5.6/lib/python3.5/site-packages/pyspark
   * Checking java version
   java version "1.8.0_152"
   Java(TM) SE Runtime Environment (build 1.8.0_152-b16)
   Java HotSpot(TM) 64-Bit Server VM (build 25.152-b16, mixed mode)
   * Testing scala command
   18/12/10 16:38:44 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
   Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
   Setting default log level to "WARN".
   To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
   18/12/10 16:38:50 WARN MetricsSystem: Using default name SparkStatusTracker for source because neither spark.metrics.namespace nor spark.app.id is set.
   18/12/10 16:39:53 WARN SparkServiceRPCClient: Now tracking server state for 5abb7c7e-df8e-4290-947c-c9a38601024e, invalidating prev state
   18/12/10 16:39:59 WARN SparkServiceRPCClient: Syncing 129 files (176036 bytes) took 3003 ms
   Welcome to
         ____              __
        / __/__  ___ _____/ /__
       _\ \/ _ \/ _ `/ __/  '_/
      /___/ .__/\_,_/_/ /_/\_\   version 2.4.0-SNAPSHOT
         /_/

   Using Scala version 2.11.12 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_152)
   Type in expressions to have them evaluated.
   Type :help for more information.

   scala> spark.range(100).reduce(_ + _)
   Spark context Web UI available at https://10.8.5.214:4040
   Spark context available as 'sc' (master = local[*], app id = local-1544488730553).
   Spark session available as 'spark'.
   View job details at <databricks-url>/?o=0#/setting/clusters/<cluster-id>/sparkUi
   View job details at <databricks-url>?o=0#/setting/clusters/<cluster-id>/sparkUi
   res0: Long = 4950

   scala> :quit

   * Testing python command
   18/12/10 16:40:16 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
   Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
   Setting default log level to "WARN".
   To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
   18/12/10 16:40:17 WARN MetricsSystem: Using default name SparkStatusTracker for source because neither spark.metrics.namespace nor spark.app.id is set.
   18/12/10 16:40:28 WARN SparkServiceRPCClient: Now tracking server state for 5abb7c7e-df8e-4290-947c-c9a38601024e, invalidating prev state
   View job details at <databricks-url>/?o=0#/setting/clusters/<cluster-id>/sparkUi
   ```

## <a name="set-up-your-ide-or-notebook-server"></a>设置 IDE 或笔记本服务器

本部分将介绍如何配置首选 IDE 或笔记本服务器来使用 Databricks Connect 客户端。

### <a name="in-this-section"></a>本节内容：

* [Jupyter](#jupyter)
* [PyCharm](#pycharm)
* [SparkR 和 RStudio Desktop](#sparkr-and-rstudio-desktop)
* [sparklyr 和 RStudio Desktop](#sparklyr-and-rstudio-desktop)
* [IntelliJ（Scala 或 Java）](#intellij-scala-or-java)
* [Eclipse](#eclipse)
* [Visual Studio Code](#visual-studio-code)
* [SBT](#sbt)

### <a name="jupyter"></a>Jupyter

Databricks Connect 配置脚本会自动将包添加到项目配置中。 若要在 Python 内核中开始操作，请运行：

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.getOrCreate()
```

若要启用 `%sql` 速记来运行和直观呈现 SQL 查询，请使用以下代码片段：

```python
from IPython.core.magic import line_magic, line_cell_magic, Magics, magics_class

@magics_class
class DatabricksConnectMagics(Magics):

   @line_cell_magic
   def sql(self, line, cell=None):
       if cell and line:
           raise ValueError("Line must be empty for cell magic", line)
       try:
           from autovizwidget.widget.utils import display_dataframe
       except ImportError:
           print("Please run `pip install autovizwidget` to enable the visualization widget.")
           display_dataframe = lambda x: x
       return display_dataframe(self.get_spark().sql(cell or line).toPandas())

   def get_spark(self):
       user_ns = get_ipython().user_ns
       if "spark" in user_ns:
           return user_ns["spark"]
       else:
           from pyspark.sql import SparkSession
           user_ns["spark"] = SparkSession.builder.getOrCreate()
           return user_ns["spark"]

ip = get_ipython()
ip.register_magics(DatabricksConnectMagics)
```

### <a name="pycharm"></a>PyCharm

Databricks Connect 配置脚本会自动将包添加到项目配置中。

#### <a name="python-3-clusters"></a>Python 3 群集

1. 转到“运行”>“编辑配置”。
2. 将 `PYSPARK_PYTHON=python3` 添加为环境变量。

   > [!div class="mx-imgBorder"]
   > ![Python 3 群集配置](../_static/images/db-connect/python3-env.png)

### <a name="sparkr-and-rstudio-desktop"></a><a id="sparkr-and-rstudio-desktop"> </a><a id="sparkr-rstudio"> </a> SparkR 和 RStudio Desktop

1. 下载[开源 Spark](https://spark.apache.org/downloads.html) 并将它解压到本地计算机。 选择与 Azure Databricks 群集 (Hadoop 2.7) 中相同的版本。
2. 运行 `databricks-connect get-jar-dir`。 此命令会返回类似 `/usr/local/lib/python3.5/dist-packages/pyspark/jars` 的路径。 复制 JAR 目录文件路径上方的一个目录的文件路径，例如 `/usr/local/lib/python3.5/dist-packages/pyspark`（即 `SPARK_HOME` 目录）。
3. 配置 Spark lib 路径和 Spark home，方式是将它们添加到 R 脚本的顶部。 将 `<spark-lib-path>` 设置为在步骤 1 中解压缩开源 Spark 包的目录。 将 `<spark-home-path>` 设置为步骤 2 中的 Databricks Connect 目录。

   ```r
   # Point to the OSS package path, e.g., /path/to/.../spark-2.4.0-bin-hadoop2.7
   library(SparkR, lib.loc = .libPaths(c(file.path('<spark-lib-path>', 'R', 'lib'), .libPaths())))

   # Point to the Databricks Connect PySpark installation, e.g., /path/to/.../pyspark
   Sys.setenv(SPARK_HOME = "<spark-home-path>")
   ```

4. 启动 Spark 会话，开始运行 SparkR 命令。

   ```r
   sparkR.session()

   df <- as.DataFrame(faithful)
   head(df)

   df1 <- dapply(df, function(x) { x }, schema(df))
   collect(df1)
   ```

### <a name="sparklyr-and-rstudio-desktop"></a><a id="sparklyr-and-rstudio-desktop"> </a><a id="sparklyr-rstudio"> </a>sparklyr 和 RStudio Desktop

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../release-notes/release-types.md)提供。

> [!NOTE]
>
> 可复制使用 Databricks Connect 在本地开发的依赖 sparklyr 的代码，并在 Azure Databricks 笔记本或在 Azure Databricks 工作区中的托管 RStudio Server 中运行该代码，只需很少的代码更改或无需代码更改即可实现。

#### <a name="in-this-section"></a>本节内容：

* [惠?](#requirements)
* [安装、配置和使用 sparklyr](#install-configure-and-use-sparklyr)
* [资源](#resources)
* [sparklyr 和 RStudio Desktop 限制](#sparklyr-and-rstudio-desktop-limitations)

#### <a name="requirements"></a>要求

* sparklyr 1.2 或更高版本。
* 带有 Databricks Connect 6.4.1 的 Databricks Runtime 6.4。

#### <a name="install-configure-and-use-sparklyr"></a>安装、配置和使用 sparklyr

1. 在 RStudio Desktop 中，从 CRAN 安装 sparklyr 1.2 或更高版本，或者从 GitHub 安装最新的主版本。

   ```r
   # Install from CRAN
   install.packages("sparklyr")

   # Or install the latest master version from GitHub
   install.packages("devtools")
   devtools::install_github("sparklyr/sparklyr")
   ```

2. 激活安装了 `databricks-connect==6.4.1` 的 Python 环境，并在终端中运行以下命令来获取 `<spark-home-path>`：

   ```bash
   databricks-connect get-spark-home
   ```

3. 启动 Spark 会话，开始运行 sparklyr 命令。

   ```r
   library(sparklyr)
   sc <- spark_connect(method = "databricks", spark_home = "<spark-home-path>")

   iris_tbl <- copy_to(sc, iris, overwrite = TRUE)

   library(dplyr)
   src_tbls(sc)

   iris_tbl %>% count
   ```

4. 关闭连接。

   ```r
   spark_disconnect(sc)
   ```

#### <a name="resources"></a>资源

有关详细信息，请参阅 sparklyr GitHub [README](https://github.com/sparklyr/sparklyr#connecting-through-databricks-connect)。

有关代码示例，请参阅 [sparklyr](../spark/latest/sparkr/sparklyr.md)。

#### <a name="sparklyr-and-rstudio-desktop-limitations"></a>sparklyr 和 RStudio Desktop 限制

不支持以下功能：

* sparklyr streaming API
* sparklyr ML API
* broom API
* csv_file serialization mode
* spark submit

### <a name="intellij-scala-or-java"></a>IntelliJ（Scala 或 Java）

1. 运行 `databricks-connect get-jar-dir`。
2. 将依赖项指向从命令返回的目录。 转到“文件”>“项目结构”>“模块”>“依赖项”>“+ 符号”>“JAR 或目录”。

   > [!div class="mx-imgBorder"]
   > ![IntelliJ JAR](../_static/images/db-connect/intelli-j-jars.png)

   为避免冲突，强烈建议从类路径中删除其他所有 Spark 安装项。 如果没法删除，请确保添加的 JAR 置于类路径的前面。 特别是，它们必须在其他所有已安装的 Spark 版本之前（否则将使用其他 Spark 版本并在本地运行，或者将引发 `ClassDefNotFoundError`）。

3. 检查 IntelliJ 中分类选项的设置。  默认设置为“全部”；如果设置调试断点，则将导致网络超时。  将其设置为“线程”，以避免停止后台网络线程。

   > [!div class="mx-imgBorder"]
   > ![IntelliJ 线程](../_static/images/db-connect/intelli-j-thread.png)

### <a name="eclipse"></a>Eclipse

1. 运行 `databricks-connect get-jar-dir`。
2. 将外部 JAR 配置指向从命令返回的目录。 转到“项目菜单”>“属性”>“Java 生成路径”>“库”>“添加外部 JAR”。

   > [!div class="mx-imgBorder"]
   > ![Eclipse 外部 JAR 配置](../_static/images/db-connect/eclipse.png)

   为避免冲突，强烈建议从类路径中删除其他所有 Spark 安装项。 如果没法删除，请确保添加的 JAR 置于类路径的前面。 特别是，它们必须在其他所有已安装的 Spark 版本之前（否则将使用其他 Spark 版本并在本地运行，或者将引发 `ClassDefNotFoundError`）。

   > [!div class="mx-imgBorder"]
   > ![Eclipse Spark 配置](../_static/images/db-connect/eclipse2.png)

### <a name="visual-studio-code"></a>Visual Studio Code

1. 验证是否已安装 [Python 扩展](https://marketplace.visualstudio.com/items?itemName=ms-python.python)。
2. 打开命令面板（在 macOS 上使用 Command+Shift+P，在 Windows/Linux 使用 Ctrl+Shift+P ）。
3. 选择 Python 解释器。 转到“代码”>“首选项”>“设置”，然后选择“Python 设置” 。
4. 运行 `databricks-connect get-jar-dir`。
5. 将从命令返回的目录添加到 `python.venvPath` 下的用户设置 JSON 中。 应将此内容添加到 Python 配置中。
6. 禁用 Linter。 在“架构”属性中**** ，然后编辑 JSON 设置   修改后的设置如下所示：

   > [!div class="mx-imgBorder"]
   > ![VS Code 配置](../_static/images/db-connect/vscode.png)

7. 如果使用虚拟环境运行（若要在 VS Code 中开发 Python，则建议使用此环境），请在命令面板中键入 `select python interpreter`，并指向与群集 Python 版本匹配的环境。

   > [!div class="mx-imgBorder"]
   > ![选择 Python 解释器](../_static/images/db-connect/select-intepreter.png)

   例如，如果群集是 Python 3.5，则本地环境也应该是 Python 3.5。

   > [!div class="mx-imgBorder"]
   > ![Python 版本](../_static/images/db-connect/python35.png)

### <a name="sbt"></a>SBT

若要使用 SBT，则必须配置 `build.sbt` 文件，使其针对 Databricks Connect JAR（而不是通常的 Spark 库依赖项）进行链接。 使用以下示例生成文件中的 `unmanagedBase` 指令执行此操作，该文件假定 Scala 应用具有 `com.example.Test` 主对象：

#### `build.sbt`

```
name := "hello-world"
version := "1.0"
scalaVersion := "2.11.6"
// this should be set to the path returned by ``databricks-connect get-jar-dir``
unmanagedBase := new java.io.File("/usr/local/lib/python2.7/dist-packages/pyspark/jars")
mainClass := Some("com.example.Test")
```

## <a name="run-examples-from-your-ide"></a>从 IDE 运行示例

### <a name="java"></a>Java

```java
import org.apache.spark.sql.SparkSession;

public class HelloWorld {

  public static void main(String[] args) {
    System.out.println("HelloWorld");
    SparkSession spark = SparkSession
      .builder()
      .master("local")
      .getOrCreate();

    System.out.println(spark.range(100).count());
    // The Spark code will execute on the Azure Databricks cluster.
  }
}
```

### <a name="python"></a>Python

```python
from pyspark.sql import SparkSession
spark = SparkSession\
.builder\
.getOrCreate()

print("Testing simple count")

# The Spark code will execute on the Azure Databricks cluster.
print(spark.range(100).count())
```

### <a name="scala"></a>Scala

```scala
import org.apache.spark.sql.SparkSession

object Test {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder()
        .master("local")
      .getOrCreate()
  println(spark.range(100).count())
  // The Spark code will execute on the Azure Databricks cluster.
  }
}
```

### <a name="work-with-dependencies"></a>使用依赖项

通常，主类或 Python 文件将具有其他依赖项 JAR 和文件。 可通过调用 `sparkContext.addJar("path-to-the-jar")` 或 `sparkContext.addPyFile("path-to-the-file")` 来添加这类依赖项 JAR 和文件。 还可使用 `addPyFile()` 接口添加 Egg 文件和 zip 文件。 每次在 IDE 中运行代码时，都会在群集上安装依赖项 JAR 和文件。

#### <a name="python"></a>Python

```python
from lib import Foo
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

sc = spark.sparkContext
#sc.setLogLevel("INFO")

print("Testing simple count")
print(spark.range(100).count())

print("Testing addPyFile isolation")
sc.addPyFile("lib.py")
print(sc.parallelize(range(10)).map(lambda i: Foo(2)).collect())

class Foo(object):
  def __init__(self, x):
    self.x = x
```

**Python + Java UDF**

```python
from pyspark.sql import SparkSession
from pyspark.sql.column import _to_java_column, _to_seq, Column

## In this example, udf.jar contains compiled Java / Scala UDFs:
#package com.example
#
#import org.apache.spark.sql._
#import org.apache.spark.sql.expressions._
#import org.apache.spark.sql.functions.udf
#
#object Test {
#  val plusOne: UserDefinedFunction = udf((i: Long) => i + 1)
#}

spark = SparkSession.builder \
  .config("spark.jars", "/path/to/udf.jar") \
  .getOrCreate()
sc = spark.sparkContext

def plus_one_udf(col):
  f = sc._jvm.com.example.Test.plusOne()
  return Column(f.apply(_to_seq(sc, [col], _to_java_column)))

sc._jsc.addJar("/path/to/udf.jar")
spark.range(100).withColumn("plusOne", plus_one_udf("id")).show()
```

#### <a name="scala"></a>Scala

```scala
package com.example

import org.apache.spark.sql.SparkSession

case class Foo(x: String)

object Test {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder()
      ...
      .getOrCreate();
    spark.sparkContext.setLogLevel("INFO")

    println("Running simple show query...")
    spark.read.parquet("/tmp/x").show()

    println("Running simple UDF query...")
    spark.sparkContext.addJar("./target/scala-2.11/hello-world_2.11-1.0.jar")
    spark.udf.register("f", (x: Int) => x + 1)
    spark.range(10).selectExpr("f(id)").show()

    println("Running custom objects query...")
    val objs = spark.sparkContext.parallelize(Seq(Foo("bye"), Foo("hi"))).collect()
    println(objs.toSeq)
  }
}
```

## <a name="access-dbutils"></a>访问 DBUtils

若要访问 `dbutils.fs` 和 `dbutils.secrets`，可使用 [Databricks 实用工具](databricks-utils.md)模块。

### <a name="python"></a>Python

```bash
pip install six
```

```python
from pyspark.dbutils import DBUtils

dbutils = DBUtils(spark.sparkContext)
print(dbutils.fs.ls("dbfs:/"))
print(dbutils.secrets.listScopes())
```

在 Python 上，若要以在本地和 Azure Databricks 群集中工作的方式访问 DBUtils 模块，请使用以下 `get_dbutils()`：

```python
def get_dbutils(spark):
  if spark.conf.get("spark.databricks.service.client.enabled") == "true":
    from pyspark.dbutils import DBUtils
    return DBUtils(spark)
  else:
    import IPython
    return IPython.get_ipython().user_ns["dbutils"]
```

### <a name="scala"></a>Scala

```scala
val dbutils = com.databricks.service.DBUtils
println(dbutils.fs.ls("dbfs:/"))
println(dbutils.secrets.listScopes())
```

### <a name="enabling-dbutilssecretsget"></a>启用 `dbutils.secrets.get`

由于安全限制，需要从工作区获取特权授权令牌才能调用 `dbutils.secrets.get`。 这与 [REST API 令牌](api/latest/authentication.md#token-management)不同，它以 `dkea...` 开头。 首次运行 `dbutils.secrets.get` 时，系统会提示你如何获取特权令牌。 使用 `dbutils.secrets.setToken(token)` 设置令牌，令牌的有效期为 48 小时。

## <a name="access-the-hadoop-filesystem"></a>访问 Hadoop 文件系统

还可使用标准 Hadoop 文件系统接口直接访问 DBFS：

```scala
> import org.apache.hadoop.fs._

// get new DBFS connection
> val dbfs = FileSystem.get(spark.sparkContext.hadoopConfiguration)
dbfs: org.apache.hadoop.fs.FileSystem = com.databricks.backend.daemon.data.client.DBFS@2d036335

// list files
> dbfs.listStatus(new Path("dbfs:/"))
res1: Array[org.apache.hadoop.fs.FileStatus] = Array(FileStatus{path=dbfs:/$; isDirectory=true; ...})

// open file
> val stream = dbfs.open(new Path("dbfs:/path/to/your_file"))
stream: org.apache.hadoop.fs.FSDataInputStream = org.apache.hadoop.fs.FSDataInputStream@7aa4ef24

// get file contents as string
> import org.apache.commons.io._
> println(new String(IOUtils.toByteArray(stream)))
```

## <a name="set-hadoop-configurations"></a>设置 Hadoop 配置

在客户端上，可使用 `spark.conf.set` API 设置 Hadoop 配置，该 API 适用于 SQL 和 DataFrame 操作。 在 `sparkContext` 上设置的 Hadoop 配置必须在群集配置中进行设置或使用笔记本。 这是因为在 `sparkContext` 上设置的配置没有绑定到用户会话，而是应用于整个群集。

## <a name="troubleshooting"></a>故障排除

运行 `databricks-connect test` 以检查连接问题。 本部分将介绍你可能遇到的一些常见问题及其各自的解决方案。

### <a name="python-version-mismatch"></a>Python 版本不匹配

检查确保在本地使用的 Python 版本至少与群集上的版本具有相同的次要版本（例如，`3.5.1` 是 `3.5.2` 是正确的，`3.5` 和 `3.6` 不正确）。

如果本地安装了多个 Python 版本，请设置 `PYSPARK_PYTHON` 环境变量（例如 `PYSPARK_PYTHON=python3`），确保 Databricks Connect 使用的版本是正确的。

### <a name="server-not-enabled"></a>未启用服务器

确保群集已使用 `spark.databricks.service.server.enabled true` 启用 Spark 服务器。 如果已启用，则应会在驱动程序日志中看到以下行：

```
18/10/25 21:39:18 INFO SparkConfUtils$: Set spark config:
spark.databricks.service.server.enabled -> true
...
18/10/25 21:39:21 INFO SparkContext: Loading Spark Service RPC Server
18/10/25 21:39:21 INFO SparkServiceRPCServer:
Starting Spark Service RPC Server
18/10/25 21:39:21 INFO Server: jetty-9.3.20.v20170531
18/10/25 21:39:21 INFO AbstractConnector: Started ServerConnector@6a6c7f42
{HTTP/1.1,[http/1.1]}{0.0.0.0:15001}
18/10/25 21:39:21 INFO Server: Started @5879ms
```

### <a name="conflicting-pyspark-installations"></a>PySpark 安装存在冲突

`databricks-connect` 包与 PySpark 冲突。 在 Python 中初始化 Spark 上下文时，安装这两种都会导致错误。 这可能以多种方式显示出来，包括“流已损坏”或“未找到类”错误。 如果已在 Python 环境中安装 PySpark，请确保先卸载它，然后再安装 databricks-connect。 卸载 PySpark 后，请确保彻底重新安装 Databricks Connect 包：

```bash
pip uninstall pyspark
pip uninstall databricks-connect
pip install -U databricks-connect==5.5.*  # or 6.*.* or 7.1.* to match your cluster version. 6.1-6.6 and 7.1 are supported
```

### <a name="conflicting-spark_home"></a>`SPARK_HOME` 存在冲突

如果你之前在计算机上使用过 Spark，则 IDE 可能配置为使用 Spark 的某个其他版本，而不是 Databricks Connect Spark。 这可能以多种方式显示出来，包括“流已损坏”或“未找到类”错误。 可检查 `SPARK_HOME` 环境变量的值来查看正在使用哪个版本的 Spark：

#### <a name="java"></a>Java

```java
System.out.println(System.getenv("SPARK_HOME"));
```

#### <a name="python"></a>Python

```python
import os
print(os.environ['SPARK_HOME'])
```

#### <a name="scala"></a>Scala

```scala
println(sys.env.get("SPARK_HOME"))
```

#### <a name="resolution"></a>解决方法

如果设置 `SPARK_HOME`，使其使用的 Spark 版本与客户端中的版本不同，则应取消设置 `SPARK_HOME` 变量，然后重试。

检查 IDE 环境变量设置，检查 `.bashrc`、`.zshrc` 或 `.bash_profile` 文件，还要检查可能设置了环境变量的其他所有位置。 你很可能必须退出再重启 IDE 来清除旧状态；如果问题仍然存在，甚至可能需要创建新项目。

无需将 `SPARK_HOME` 设置为新值；取消设置就已足够。

### <a name="conflicting-or-missing-path-entry-for-binaries"></a>二进制文件的 `PATH` 项冲突或缺失

可能是这样配置 PATH 的：`spark-shell` 等命令将运行之前安装的其他某些二进制文件，而不是运行随附 Databricks Connect 提供的二进制文件。 这可能会导致 `databricks-connect test` 失败。 应确保 Databricks Connect 二进制文件优先，或者删除之前安装的二进制文件。

如果无法运行 `spark-shell` 之类的命令，也可能是 `pip install` 未自动设置 PATH，而且你需要将 `bin` dir 手动安装到 PATH 中。 即使未设置此项，也可将 Databricks Connect 与 IDE 一起使用。 但是，`databricks-connect test` 命令将不起作用。

### <a name="conflicting-serialization-settings-on-the-cluster"></a>群集上的序列化设置存在冲突

如果在运行 `databricks-connect test` 时看到“流已损坏”错误，这可能是由于群集序列化配置不兼容而造成的。 例如，设置 `spark.io.compression.codec` 配置可能会导致此问题。 若要解决此问题，请考虑从群集设置中删除这些配置，或在 Databricks Connect 客户端中设置配置。

### <a name="cannot-find-winutilsexe-on-windows"></a>在 Windows 上找不到 winutils.exe

如果正在 Windows 上使用 Databricks Connect，请参阅：

```
ERROR Shell: Failed to locate the winutils binary in the hadoop binary path
java.io.IOException: Could not locate executable null\bin\winutils.exe in the Hadoop binaries.
```

按照说明[在 Windows 上配置 Hadoop 路径](https://cwiki.apache.org/confluence/display/HADOOP2/Hadoop2OnWindows)。

### <a name="the-filename-directory-name-or-volume-label-syntax-is-incorrect-on-windows"></a>Windows 上的文件名、目录名称或卷标签语法不正确

如果正在 Windows 上使用 Databricks Connect，请参阅：

```
The filename, directory name, or volume label syntax is incorrect.
```

Java 或 Databricks Connect 已安装到[路径中带有空格](https://stackoverflow.com/questions/47028892/why-does-spark-shell-fail-with-the-filename-directory-name-or-volume-label-sy)的目录中。 要解决此问题，可安装到不带空格的目录路径或使用[短名称格式](https://stackoverflow.com/questions/892555/how-do-i-specify-c-program-files-without-a-space-in-it-for-programs-that-cant)配置路径。

## <a name="limitations"></a>限制

不支持以下 Azure Databricks 功能和第三方平台：

* 以下 [Databricks 实用工具](databricks-utils.md)：[库](databricks-utils.md#dbutils-library)、[笔记本工作流](databricks-utils.md#dbutils-workflow)和[小组件](databricks-utils.md#dbutils-widgets)。
* 结构化流。
* 在远程群集上运行未包含在 Spark 作业中的任意代码。
* 用于 Delta 表操作的本机 Scala、Python 和 R API（例如 `DeltaTable.forPath`）。 但是，具有 Delta Lake 操作的 SQL API (`spark.sql(...)`) 和 Delta 表上的常规 Spark API（例如 `spark.read.load`）都受支持。
* [Apache Zeppelin](https://zeppelin.apache.org/) 0.7.x 及更低版本。