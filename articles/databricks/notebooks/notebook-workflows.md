---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 06/08/2020
title: 笔记本工作流 - Azure Databricks
description: 了解如何实现笔记本工作流，使你能够从笔记本返回值并生成使用依赖项的复杂工作流和管道。
ms.openlocfilehash: 9c366e9c3288cd60f95758c57d1acdc0e6026d27
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106754"
---
# <a name="notebook-workflows"></a>笔记本工作流

使用 [%run](notebooks-use.md#run) 命令，可在笔记本中包含另一个笔记本。 通过此命令，你可连接表示关键 ETL 步骤、Spark 分析步骤或即席探索的各种笔记本。 但是，它不能生成更复杂的数据管道。

笔记本工作流是 `%run` 的补充，因为它们允许从笔记本返回值。 这使你可以轻松地生成包含依赖项的复杂工作流和管道。 可以正确地对运行进行参数化（例如，获取目录中的文件列表并将名称传递到另一个笔记本，这些操作无法通过 `%run` 完成），还可根据返回值创建 if/then/else 工作流。 借助笔记本工作流，可通过相对路径调用其他笔记本。

可使用 `dbutils.notebook` 方法实现笔记本工作流。 这些方法（如所有 `dbutils` API）仅适用于 Scala 和 Python。 但可使用 `dbutils.notebook.run` 调用 R 笔记本。

> [!NOTE]
>
> 不支持长时间运行（完成时间超过 48 小时）的笔记本工作流[作业](../jobs.md)。

## <a name="api"></a>API

`dbutils.notebook` API 中可用于生成笔记本工作流的方法包括：`run` 和 `exit`。 参数和返回值都必须是字符串。

**`run(path: String,  timeout_seconds: int, arguments: Map): String`**

运行笔记本并返回其退出值。 该方法会启动一个立即运行的临时作业。

`timeout_seconds` 参数控制运行的超时值（0 表示无超时）：如果对 `run` 的调用在指定时间内未完成，则会引发异常。 如果 Azure Databricks 停机时间超过 10 分钟，笔记本运行将失败，而不考虑 `timeout_seconds`。

`arguments` 参数可设置目标笔记本的小组件值。 具体而言，如果正在运行的笔记本具有名为 `A` 的小组件，而且你将键值对 `("A": "B")` 作为 arguments 参数的一部分传递给 `run()` 调用，则检索小组件 `A` 的值将返回 `"B"`。 可在[小组件](widgets.md)一文中找到有关创建和使用小组件的说明。

> [!WARNING]
>
> `arguments` 参数只接受拉丁字符（ASCII 字符集）。 使用非 ASCII 字符将会返回错误。 例如，中文、日文汉字和表情符号都属于无效的非 ASCII 字符。

### <a name="run-usage"></a>`run` 用法

#### <a name="python"></a>Python

```python
dbutils.notebook.run("notebook-name", 60, {"argument": "data", "argument2": "data2", ...})
```

#### <a name="scala"></a>Scala

```scala
dbutils.notebook.run("notebook-name", 60, Map("argument" -> "data", "argument2" -> "data2", ...))
```

### <a name="run-example"></a>`run` 示例

假设你有一个名为 `workflows` 的笔记本，其中包含一个名为 `foo`，该笔记本将小组件值打印为：

```scala
dbutils.widgets.text("foo", "fooDefault", "fooEmptyLabel")
print dbutils.widgets.get("foo")
```

运行 `dbutils.notebook.run("workflows", 60, {"foo": "bar"})` 将产生以下结果：

> [!div class="mx-imgBorder"]
> ![包含小组件的笔记本工作流](../_static/images/notebooks/notebook-workflow-widget-example.png)

小组件具有通过工作流传入的值 `"bar"`，而不是默认值。

`exit(value: String): void` 使用值退出笔记本。 如果使用 `run` 方法调用笔记本，则会返回以下值。

```scala
dbutils.notebook.exit("returnValue")
```

在作业中调用 `dbutils.notebook.exit` 可导致笔记本成功完成。 如果希望作业失败，请引发异常。

## <a name="example"></a><a id="example"> </a><a id="notebook-workflows-exit"> </a>示例

以下示例将 arguments 传递到 `DataImportNotebook` 并根据 `DataImportNotebook` 的结果运行不同的笔记本（`DataCleaningNotebook` 或 `ErrorHandlingNotebook`）。

> [!div class="mx-imgBorder"]
> ![笔记本工作流](../_static/images/notebooks/notebook-workflow-example.png)

当笔记本工作流运行时，会显示指向正在运行的笔记本的链接：

> [!div class="mx-imgBorder"]
> ![笔记本工作流运行](../_static/images/notebooks/dbutils.run.png)

单击“笔记本作业 #xxxx”笔记本链接，查看该运行的详细信息：

> [!div class="mx-imgBorder"]
> ![笔记本工作流运行结果](../_static/images/notebooks/notebook-run-results.png)

## <a name="pass-structured-data"></a>传递结构化数据

本部分说明如何在笔记本之间传递结构化数据。

### <a name="python"></a>Python

```python
# Example 1 - returning data through temporary views.
# You can only return one string using dbutils.notebook.exit(), but since called notebooks reside in the same JVM, you can
# return a name referencing data stored in a temporary view.

## In callee notebook
sqlContext.range(5).toDF("value").createOrReplaceGlobalTempView("my_data")
dbutils.notebook.exit("my_data")

## In caller notebook
returned_table = dbutils.notebook.run("LOCATION_OF_CALLEE_NOTEBOOK", 60)
global_temp_db = spark.conf.get("spark.sql.globalTempDatabase")
display(table(global_temp_db + "." + returned_table))

# Example 2 - returning data through DBFS.
# For larger datasets, you can write the results to DBFS and then return the DBFS path of the stored data.

## In callee notebook
dbutils.fs.rm("/tmp/results/my_data", recurse=True)
sqlContext.range(5).toDF("value").write.parquet("dbfs:/tmp/results/my_data")
dbutils.notebook.exit("dbfs:/tmp/results/my_data")

## In caller notebook
returned_table = dbutils.notebook.run("LOCATION_OF_CALLEE_NOTEBOOK", 60)
display(sqlContext.read.parquet(returned_table))

# Example 3 - returning JSON data.
# To return multiple values, you can use standard JSON libraries to serialize and deserialize results.

## In callee notebook
import json
dbutils.notebook.exit(json.dumps({
  "status": "OK",
  "table": "my_data"
}))

## In caller notebook
result = dbutils.notebook.run("LOCATION_OF_CALLEE_NOTEBOOK", 60)
print json.loads(result)
```

### <a name="scala"></a>Scala

```scala
// Example 1 - returning data through temporary views.
// You can only return one string using dbutils.notebook.exit(), but since called notebooks reside in the same JVM, you can
// return a name referencing data stored in a temporary view.

/** In callee notebook */
sc.parallelize(1 to 5).toDF().createOrReplaceGlobalTempView("my_data")
dbutils.notebook.exit("my_data")

/** In caller notebook */
val returned_table = dbutils.notebook.run("LOCATION_OF_CALLEE_NOTEBOOK", 60)
val global_temp_db = spark.conf.get("spark.sql.globalTempDatabase")
display(table(global_temp_db + "." + returned_table))

// Example 2 - returning data through DBFS.
// For larger datasets, you can write the results to DBFS and then return the DBFS path of the stored data.

/** In callee notebook */
dbutils.fs.rm("/tmp/results/my_data", recurse=true)
sc.parallelize(1 to 5).toDF().write.parquet("dbfs:/tmp/results/my_data")
dbutils.notebook.exit("dbfs:/tmp/results/my_data")

/** In caller notebook */
val returned_table = dbutils.notebook.run("LOCATION_OF_CALLEE_NOTEBOOK", 60)
display(sqlContext.read.parquet(returned_table))

// Example 3 - returning JSON data.
// To return multiple values, you can use standard JSON libraries to serialize and deserialize results.

/** In callee notebook */

// Import jackson json libraries
import com.fasterxml.jackson.module.scala.DefaultScalaModule
import com.fasterxml.jackson.module.scala.experimental.ScalaObjectMapper
import com.fasterxml.jackson.databind.ObjectMapper

// Create a json serializer
val jsonMapper = new ObjectMapper with ScalaObjectMapper
jsonMapper.registerModule(DefaultScalaModule)

// Exit with json
dbutils.notebook.exit(jsonMapper.writeValueAsString(Map("status" -> "OK", "table" -> "my_data")))

/** In caller notebook */
val result = dbutils.notebook.run("LOCATION_OF_CALLEE_NOTEBOOK", 60)
println(jsonMapper.readValue[Map[String, String]](result))
```

## <a name="handle-errors"></a>处理错误

本部分说明如何处理笔记本工作流中的错误。

### <a name="python"></a>Python

```python
# Errors in workflows thrown a WorkflowException.

def run_with_retry(notebook, timeout, args = {}, max_retries = 3):
  num_retries = 0
  while True:
    try:
      return dbutils.notebook.run(notebook, timeout, args)
    except Exception as e:
      if num_retries > max_retries:
        raise e
      else:
        print "Retrying error", e
        num_retries += 1

run_with_retry("LOCATION_OF_CALLEE_NOTEBOOK", 60, max_retries = 5)
```

### <a name="scala"></a>Scala

```scala
// Errors in workflows thrown a WorkflowException.

import com.databricks.WorkflowException

// Since dbutils.notebook.run() is just a function call, you can retry failures using standard Scala try-catch
// control flow. Here we show an example of retrying a notebook a number of times.
def runRetry(notebook: String, timeout: Int, args: Map[String, String] = Map.empty, maxTries: Int = 3): String = {
  var numTries = 0
  while (true) {
    try {
      return dbutils.notebook.run(notebook, timeout, args)
    } catch {
      case e: WorkflowException if numTries < maxTries =>
        println("Error, retrying: " + e)
    }
    numTries += 1
  }
  "" // not reached
}

runRetry("LOCATION_OF_CALLEE_NOTEBOOK", timeout = 60, maxTries = 5)
```

## <a name="run-multiple-notebooks-concurrently"></a>同时运行多个笔记本

可使用标准 Scala 和 Python 构造同时运行多个笔记本，如 Thread（[Scala](https://docs.oracle.com/javase/7/docs/api/java/lang/Thread.html) 和 [Python](https://docs.python.org/2/library/threading.html)）和 Future（[Scala](https://docs.scala-lang.org/overviews/core/futures.html) 和 [Python](https://docs.python.org/2/library/multiprocessing.html)）。 高级笔记本工作流笔记本演示如何使用这些构造。 笔记本为 Scala 语言，但你可轻松使用 Python 编写等效内容。 运行示例：

1. 下载[笔记本存档](../_static/notebooks/advanced-notebook-workflows.dbc)。
2. 将存档导入到工作区。
3. 运行“Concurrent Notebooks”笔记本。