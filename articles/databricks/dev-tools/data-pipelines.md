---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 管理数据管道中的依赖关系 - Azure Databricks
description: 了解在将数据管道与 Azure Databricks 配合使用时如何管理依赖关系。
ms.openlocfilehash: 805bff9b8239638a2e71d1c930df97dd402c0682
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937668"
---
# <a name="managing-dependencies-in-data-pipelines"></a>管理数据管道中的依赖项

数据管道中通常存在复杂的依赖关系。 可以通过工作流系统描述此类依赖关系，并在管道运行时进行计划。

## <a name="azure-data-factory"></a>Azure 数据工厂

[Azure 数据工厂](/data-factory/)是一项云数据集成服务，可用于将数据存储、移动和处理服务组合到自动化数据管道中。 可以在 Azure 数据工厂数据管道中使 Databricks 笔记本可操作。 请参阅[在 Azure 数据工厂中使用 Databricks 笔记本活动运行 Databricks 笔记本](/data-factory/transform-data-using-databricks-notebook)，了解如何创建在 Azure Databricks 群集中运行 Databricks 笔记本的 Azure 数据工厂管道，然后[通过运行 Databricks 笔记本来转换数据](/data-factory/transform-data-databricks-notebook)。

## <a name="apache-airflow"></a><a id="airflow"> </a><a id="apache-airflow"> </a>Apache Airflow

[Apache Airflow](https://airflow.incubator.apache.org/) 是一种用于管理和计划数据管道的解决方案。 Airflow 将数据管道表示为操作的有向无环图 (DAG)，其中，一条边表示操作之间的一个逻辑依赖关系。

Airflow 提供 Azure Databricks 与 Airflow 之间的紧密集成。 可以通过 Airflow Azure Databricks 集成将 Azure Databricks 提供的已优化的 Spark 引擎与 Airflow 的计划功能配合使用。

### <a name="install-the-airflow-azure-databricks-integration"></a>安装 Airflow Azure Databricks 集成

Airflow 与 Azure Databricks 的集成在 Airflow 1.9.0 版中提供。 若要安装 Airflow Azure Databricks 集成，请运行：

```bash
pip install "apache-airflow[databricks]"
```

若要安装[额外项](https://airflow.incubator.apache.org/installation.html?highlight=extras#extra-packages)（例如 `celery` 和 `password`），请运行：

```bash
pip install "apache-airflow[databricks, celery, password]"
```

### <a name="databricksrunnowoperator-operator"></a>`DatabricksRunNowOperator` 运算符

Airflow Azure Databricks 集成提供 [DatabricksRunNowOperator](https://airflow.readthedocs.io/en/latest/integration.html#databricks) 作为计算的 DAG 中的一个节点。 此运算符与 Databricks 作业[立即运行](api/latest/jobs.md#jobsjobsservicerunnow) API 终结点匹配，允许你以编程方式运行上传到 DBFS 的笔记本和 JAR。

### <a name="databrickssubmitrunoperator-operator"></a>`DatabricksSubmitRunOperator` 运算符

Airflow Azure Databricks 集成提供 [DatabricksSubmitRunOperator](https://airflow.readthedocs.io/en/latest/integration.html#databricks) 作为计算的 DAG 中的一个节点。 此运算符与 Databricks 作业[提交运行](api/latest/jobs.md#jobsjobsservicesubmitrun) API 终结点匹配，允许你以编程方式运行上传到 DBFS 的笔记本和 JAR。

### <a name="configure-a-databricks-connection"></a><a id="airflow-connection"> </a><a id="configure-a-databricks-connection"> </a>配置 Databricks 连接

若要使用 `DatabricksSubmitRunOperator`，必须在相应的 Airflow 连接中提供凭据。 默认情况下，如果未为 `DatabricksSubmitRunOperator` 指定 `databricks_conn_id` 参数，该运算符会尝试在 ID 为 `databricks_default` 的连接中查找凭据。

可以按照[管理连接](https://airflow.readthedocs.io/en/stable/howto/connection/index.html)中的说明通过 Airflow Web UI 配置 Airflow 连接。 对于 Databricks 连接，请将“主机”字段设置为 Databricks 部署的主机名，将“登录名”字段设置为 `token`，将“密码”字段设置为 Databricks 生成的[个人访问令牌](api/latest/authentication.md)，并将“额外项”字段设置为

```json
{"token": "<your personal access token>"}
```

### <a name="example"></a>示例

在此示例中，我们演示如何设置一个在本地计算机上运行的简单的 Airflow 部署，并部署一个已命名的示例 DAG，用于在 Databricks 中触发运行。

#### <a name="initialize-airflow-database"></a>初始化 Airflow 数据库

初始化可供 Airflow 用来跟踪其他元数据的 SQLite 数据库。 在生产型 Airflow 部署中，可以为 Airflow 配置一个标准数据库。 若要执行初始化运行，请执行以下语句：

```bash
airflow initdb
```

适用于 Airflow 部署的 SQLite 数据库和默认配置在 `~/airflow` 中初始化。

#### <a name="dag-definition"></a>DAG 定义

DAG 定义是一个 Python 文件，在此示例中命名为 `example_databricks_operator.py`。 此示例运行两个具有一种线性依赖关系的 Databricks 作业。 第一个 Databricks 作业触发位于 `/Users/airflow@example.com/PrepareData` 的笔记本，第二个将运行位于 `dbfs:/lib/etl-0.1.jar` 的 JAR。 示例 DAG 定义构造两个 `DatabricksSubmitRunOperator` 任务，然后通过 `set_dowstream` 方法在结尾处设置依赖关系。 代码的主干版本如下所示：

```python
notebook_task = DatabricksSubmitRunOperator(
    task_id='notebook_task',
    dag=dag,
    json=notebook_task_params)

spark_jar_task = DatabricksSubmitRunOperator(
    task_id='spark_jar_task',
    dag=dag,
    json=spark_jar_task_params)

notebook_task.set_downstream(spark_jar_task)
```

##### <a name="import-airflow-and-required-classes"></a>导入 Airflow 和所需的类

DAG 定义的顶部导入了 `airflow`、`DAG` 和 `DatabricksSubmitRunOperator`：

```python
import airflow

from airflow import DAG
from airflow.contrib.operators.databricks_operator import DatabricksSubmitRunOperator
```

##### <a name="configure-global-arguments"></a>配置全局参数

下一节设置应用于 DAG 中的每个任务的默认参数。

```python
args = {
    'owner': 'airflow',
    'email': ['airflow@example.com'],
    'depends_on_past': False,
    'start_date': airflow.utils.dates.days_ago(0)
}
```

两个值得一提的参数是 `depends_on_past` 和 `start_date`。 将 `depends_on_past` 设置为 `true` 意味着，除非任务的上一个实例成功完成，否则不应触发该任务。 `start_date argument` 决定了第一个任务实例计划在何时执行。

##### <a name="instantiate-the-dag"></a>实例化 DAG

DAG 实例化语句为 DAG 提供唯一 ID，附加默认参数，并为其提供每日计划。

```python
dag = DAG(dag_id='example_databricks_operator', default_args=args, schedule_interval='@daily')
```

下一语句指定要运行任务的群集中的 Spark 版本、节点类型和工作器数。 规范的架构与作业[提交运行](api/latest/jobs.md#jobsjobsservicesubmitrun)终结点的 `new_cluster` 字段匹配。

```python
new_cluster = {
    'spark_version': '6.0.x-scala2.11',
    "node_type_id": "Standard_D3_v2",
    'num_workers': 8
}
```

##### <a name="register-tasks-in-dag"></a>在 DAG 中注册任务

对于 `notebook_task`，请将 `DatabricksSubmitRunOperator` 实例化。

```python
notebook_task_params = {
    'new_cluster': new_cluster,
    'notebook_task': {
    'notebook_path': '/Users/airflow@example.com/PrepareData',
  },
}
# Example of using the JSON parameter to initialize the operator.
notebook_task = DatabricksSubmitRunOperator(
  task_id='notebook_task',
  dag=dag,
  json=notebook_task_params)
```

在此代码段中，JSON 参数采用与 `Runs Submit` 终结点匹配的 Python 字典。

对于 `spark_jar_task`（运行位于 `dbfs:/lib/etl-0.1.jar` 的 JAR），请将 `DatabricksSubmitRunOperator` 实例化。

```python
# Example of using the named parameters of DatabricksSubmitRunOperator to initialize the operator.
spark_jar_task = DatabricksSubmitRunOperator(
  task_id='spark_jar_task',
  dag=dag,
  new_cluster=new_cluster,
  spark_jar_task={
    'main_class_name': 'com.example.ProcessData'
  },
  libraries=[
    {
      'jar': 'dbfs:/lib/etl-0.1.jar'
    }
  ]
)
```

若要配置 `spark_jar_task` 以运行下游项目，请使用 `notebook_task` 上的 `set_downstream` 方法来注册依赖关系。

```python
notebook_task.set_downstream(spark_jar_task)
```

请注意，在 `notebook_task` 中，我们已使用 `json` 参数指定“提交运行”终结点的完整规范，而在 `spark_jar_task` 中，我们已将“提交运行”终结点的顶级键平展为 `DatabricksSubmitRunOperator` 的参数。 尽管两种实例化运算符的方法是等效的，但后一方法不允许使用任何新的顶级字段，如 `spark_python_task` 或 `spark_submit_task`。 有关详细信息，请参阅 [DatabricksSubmitRunOperator API](https://airflow.readthedocs.io/en/latest/integration.html#databrickssubmitrunoperator)。

#### <a name="install-and-verify-the-dag-in-airflow"></a>在 Airflow 中安装 DAG 并进行验证

若要在 Airflow 中安装 DAG，请创建目录 `~/airflow/dags`，然后将 DAG 定义文件复制到该目录中。

若要验证 Airflow 是否已读入 DAG，请运行 `list_dags` 命令：

```bash
airflow list_dags

[2017-07-06 10:27:23,868] {__init__.py:57} INFO - Using executor SequentialExecutor
[2017-07-06 10:27:24,238] {models.py:168} INFO - Filling up the DagBag from /Users/<user>/airflow/dags

-------------------------------------------------------------------
DAGS
-------------------------------------------------------------------
example_bash_operator
example_branch_dop_operator_v3
example_branch_operator
example_databricks_operator
...
```

#### <a name="visualize-the-dag-in-the-airflow-ui"></a>在 Airflow UI 中可视化 DAG

可以在 Airflow Web UI 中可视化 DAG。 运行 `airflow webserver` 并连接到 `localhost:8080`。 单击 `example_databricks_operator` 以查看 DAG 的多个可视化效果。 以下是示例：

> [!div class="mx-imgBorder"]
> ![Airflow DAG](../_static/images/jobs/airflow-tutorial.png)

#### <a name="configure-the-connection-to-airflow"></a>配置与 Airflow 的连接

DAG 定义中未指定 Databricks 的连接凭据。 默认情况下，`DatabricksSubmitRunOperator` 将 `databricks_conn_id` 参数设置为 `databricks_default`，因此，请使用[配置 Databricks 连接](#airflow-connection)中所述的 Web UI 为 ID `databricks_default` 添加一个连接。

#### <a name="test-each-task"></a>测试每个任务

若要测试 `notebook_task`，请运行 `airflow test example_databricks_operator notebook_task <YYYY-MM-DD>`；对于 `spark_jar_task`，请运行 `airflow test example_databricks_operator spark_jar_task <YYYY-MM-DD>`。 若要按计划运行 DAG，可使用命令 `airflow scheduler` 调用计划程序后台进程。

启动计划程序后，应该能够在 Web UI 中看到 DAG 的回填运行。