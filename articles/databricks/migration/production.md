---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 将生产工作负载迁移到 Azure Databricks - Azure Databricks
description: 了解如何将生产 Apache Spark 作业迁移到 Azure Databricks。
ms.openlocfilehash: ca47c4e27655b5b62ae0adc371da1e9ec0514afd
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937759"
---
# <a name="migrate-production-workloads-to-azure-databricks"></a>将生产工作负荷迁移到 Azure Databricks

本指南说明如何将生产作业从其他平台上的 Apache Spark 移到 Azure Databricks 上的 Apache Spark。

## <a name="concepts"></a>概念

**[Databricks 作业](../jobs.md)**

可以捆绑并提交到 Azure Databricks 的单个代码单元。 Azure Databricks 作业等效于具有单个 `SparkContext` 的 [Spark 应用程序](https://spark.apache.org/docs/latest/submitting-applications)。 入口点可以位于库（例如 JAR、egg、wheel）或笔记本中。 你可以使用复杂的重试和警报机制按计划运行 Azure Databricks 作业。 运行作业的主要界面为作业 [API](../dev-tools/api/latest/jobs.md) 和 [UI](../jobs.md#job-create)。

**[池](../clusters/instance-pools/index.md)**

帐户中的一组实例，由 Azure Databricks 管理，但在空闲时不会产生任何 Azure Databricks 费用。
在池上提交多个作业可以确保作业快速启动。 可以为实例池设置护栏（实例类型、实例限制等）和自动缩放策略。 池等效于其他 Spark 平台上的自动缩放群集。

## <a name="migration-steps"></a>迁移步骤

本部分提供将生产作业移到 Azure Databricks 的步骤。

### <a name="step-1-create-a-pool"></a>步骤 1：创建池

[创建自动缩放池](../clusters/instance-pools/create.md#instance-pools-create)。
这等效于在其他 Spark 平台中创建自动缩放群集。 在其他平台上，如果自动缩放群集中的实例在几分钟或几小时内处于空闲状态，则需要为其付费。 Azure Databricks 为你免费管理实例池。 也就是说，如果这些计算机未处于使用状态，则无需支付 Azure Databricks 费用；你只需要向云提供商付费。 仅当在实例上运行作业时，Azure Databricks 才收费。

> [!div class="mx-imgBorder"]
> ![创建池](../_static/images/migration/pool.png)

关键配置：

* **最小空闲**：池维护的未被作业使用的备用实例数。 可将其设置为 0。
* **最大容量**：这是一个可选字段。 如果已设置云提供商实例限制，则可以将此字段留空。
  如果要设置其他最大限制，请设置一个较高的值，以便大量作业可以共享该池。
* **空闲实例自动终止**：如果设置了“最小空闲”的实例在指定时间段内处于空闲状态，则将其释放回云提供商。 值越高，实例保持准备就绪状态的时间越长，因而作业的启动速度就越快。

### <a name="step-2-run-a-job-on-a-pool"></a>步骤 2：在池上运行作业

可使用作业 API 或 UI 在池上运行作业。 必须通过提供群集规范来运行每个作业。当作业将要启动时，Azure Databricks 会自动从池中创建新群集。 作业完成后，群集将自动终止。 完全按照作业的运行时间向你收费。 这是在 Azure Databricks 上运行作业最具成本效益的方法。 每个新群集均具有：

* 一个关联的 `SparkContext`，其等效于其他 Spark 平台上的 Spark 应用程序。
* 一个驱动程序节点和指定数量的辅助角色。 对于单个作业，可以指定辅助角色范围。 Azure Databricks 基于单个 Spark 作业所需的资源自动缩放该作业。 Azure Databricks [基准测试](https://databricks.com/blog/2018/05/02/introducing-databricks-optimized-auto-scaling)表明，根据作业的性质，这可以为你节省最多 30% 的云成本。

在池上运行作业的方法有以下三种：API/CLI、Airflow、UI。

#### <a name="api--cli"></a>API/CLI

1. 下载并配置 [Databricks CLI](../dev-tools/cli/index.md)。
2. 运行以下命令来提交一次代码。 API 将返回一个 URL，你可以使用该 URL 跟踪作业运行的进度。

   ```bash
   databricks runs submit --json

   {
     "run_name": "my spark job",
     "new_cluster": {
       "spark_version": "5.0.x-scala2.11",

       "instance_pool_id": "0313-121005-test123-pool-ABCD1234"**,**
       "num_workers": 10
       },
       "libraries": [
       {
       "jar": "dbfs:/my-jar.jar"
       }

       ],
       "timeout_seconds": 3600,
       "spark_jar_task": {
       "main_class_name": "com.databricks.ComputeModels"
     }
   }
   ```

3. 若要计划作业，请使用下面的示例。 通过此机制创建的作业将显示在[作业列表页](../jobs.md#view-jobs)中。 返回值为 `job_id`，可用于查看所有运行的状态。

   ```bash
   databricks jobs create --json

   {
     "name": "Nightly model training",
     "new_cluster": {
        "spark_version": "5.0.x-scala2.11",
        ...
        **"instance_pool_id": "0313-121005-test123-pool-ABCD1234",**
        "num_workers": 10
      },
      "libraries": [
        {
        "jar": "dbfs:/my-jar.jar"
        }
      ],
      "email_notifications": {
        "on_start": ["john@foo.com"],
        "on_success": ["sally@foo.com"],
        "on_failure": ["bob@foo.com"]
      },
      "timeout_seconds": 3600,
      "max_retries": 2,
      "schedule": {
      "quartz_cron_expression": "0 15 22 ? \* \*",
      "timezone_id": "America/Los_Angeles"
      },
      "spark_jar_task": {
        "main_class_name": "com.databricks.ComputeModels"
     }
   }
   ```

如果使用 [spark-submit](https://spark.apache.org/docs/latest/submitting-applications#launching-applications-with-spark-submit) 提交 Spark 作业，则下表显示了 spark-submit 参数如何映射到[作业创建 API](../dev-tools/api/latest/jobs.md#jobsjobsservicecreatejob) 中的不同参数。

| spark-submit 参数                 | 如何将其应用于 Azure Databricks                                                                                                                                                                         |
|----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| –class                                 | 使用 [Spark JAR 任务](../dev-tools/api/latest/jobs.md#jobssparkjartask)提供主类名和参数。                                                                              |
| –jars                                  | 使用 `libraries` 参数提供依赖项列表。                                                                                                                                          |
| –py-files                              | 对于 Python 作业，请使用 [Spark Python 任务](../dev-tools/api/latest/jobs.md#jobssparkpythontask)。 可使用 `libraries` 参数来提供 egg 或 wheel 依赖项。                              |
| –master                                | 在云中，无需管理长时间运行的主节点。 所有实例和作业均由 Azure Databricks 服务管理。 忽略此参数。                                             |
| –deploy-mode                           | 忽略此 Azure Databricks 上的参数。                                                                                                                                                                 |
| –conf                                  | 在 [NewCluster 规范](../dev-tools/api/latest/jobs.md#jobsclusterspecnewcluster)中，使用 `spark_conf` 参数。                                                                                        |
| –num-executors                         | 在 [NewCluster 规范](../dev-tools/api/latest/jobs.md#jobsclusterspecnewcluster)中，使用 `num_workers` 参数。 你还可以使用 `autoscale` 选项来提供一个范围（推荐）。             |
| –driver-memory、–driver-cores          | 根据所需的驱动程序内存和核心，选择适当的实例类型。                                                                                                                        |
|                                        | 你将在池创建期间为驱动程序提供实例类型。 在作业提交过程中忽略此参数。                                                                                   |
| –executor-memory、–executor-cores      | 根据所需的执行程序内存，选择适当的实例类型。                                                                                                                                |
|                                        | 你将在池创建期间为辅助角色提供实例类型。 在作业提交过程中忽略此参数。                                                                                  |
| –driver-class-path                     | 在 spark_conf 参数中，将 `spark.driver.extraClassPath` 设置为合适的值。                                                                                                                         |
| –driver-java-options                   | 在 spark_conf 参数中，将 `spark.driver.extraJavaOptions` 设置为合适的值。                                                                                                                   |
| –files                                 | 在 `spark_conf` 参数中，将 `spark.files` 设置为合适的值。                                                                                                                                   |
| –name                                  | 在[运行提交请求](../dev-tools/api/latest/jobs.md#jobssubmitrun)中，使用 run_name 参数。 在[创建作业请求](../dev-tools/api/latest/jobs.md#jobscreatejob)中，使用 name 参数。 |

#### <a name="airflow"></a>气流

如果想要使用 Airflow 在 Azure Databricks 中提交作业，Azure Databricks 提供 [Airflow 运算符](https://airflow.readthedocs.io/en/stable/integration.html?#databricks)。 Databricks Airflow 运算符调用[作业运行 API](../dev-tools/api/latest/jobs.md#jobsjobsservicerunnow) 将作业提交到 Azure Databricks。
请参阅 [Apache Airflow](../dev-tools/data-pipelines.md#airflow)。

#### <a name="ui"></a>UI

Azure Databricks 提供了一种简单直观易用的 UI 来提交和计划作业。 若要通过 UI 创建和提交作业，请遵循[分步指南](../jobs.md#job-create)。

### <a name="step-3-troubleshoot-jobs"></a>步骤 3：作业故障排除

Azure Databricks 提供了许多工具来帮助你对作业进行故障排除。

#### <a name="access-logs-and-spark-ui"></a>访问日志和 Spark UI

Azure Databricks 维护完全托管的 Spark 历史记录服务器，使你可以访问每个作业运行的所有 Spark 日志和 Spark UI。 可从[作业详细信息页](../jobs.md#job-details)以及作业运行页访问它们：

> [!div class="mx-imgBorder"]
> ![作业运行](../_static/images/migration/job.png)

#### <a name="forward-logs"></a>转发日志

你还可以将群集日志转发到你的云存储位置。 若要将日志发送到所选位置，请使用 [NewCluster 规范](../dev-tools/api/latest/jobs.md#jobsclusterspecnewcluster)中的 `cluster_log_conf` 参数。

#### <a name="view-metrics"></a>查看指标

作业运行时，你可以转到群集页，然后在“指标”选项卡中查看实时 Ganglia 指标。Azure Databricks 还会每 15 分钟对这些指标进行一次快照并将其存储，因此即使在作业完成后，你也可以查看这些指标。 若要将指标发送到指标服务器，可以在群集中安装自定义代理。
请参阅[监视性能](../clusters/clusters-manage.md#cluster-performance)。

> [!div class="mx-imgBorder"]
> ![Ganglia 指标](../_static/images/migration/metrics.png)

#### <a name="set-alerts"></a>设置警报

使用[作业创建 API](../dev-tools/api/latest/jobs.md#jobsjobsservicecreatejob) 中的 email_notifications 获取有关作业失败的警报。
你还可以将这些电子邮件警报转发给 PagerDuty、Slack 和其他监视系统。

* [如何通过电子邮件设置 PagerDuty 警报](https://www.pagerduty.com/docs/guides/email-integration-guide/)
* [如何通过电子邮件设置 Slack 通知](https://get.slack.help/hc/articles/206819278-Send-emails-to-Slack)

## <a name="frequently-asked-questions-faqs"></a>常见问题 (FAQ)

**能否不使用池运行作业？**

能。 池是可选的。 可以直接在新群集上运行作业。 在这种情况下，Azure Databricks 会通过向云提供商要求所需实例来创建群集。 对于池，如果池中实例可用，则群集启动时间将为约 30 秒。

**什么是笔记本作业？**

Azure Databricks 具有不同的作业类型 - JAR、Python、笔记本。 笔记本作业类型在指定的笔记本中运行代码。 请参阅[笔记本作业提示](../jobs.md#notebook-jobs)。

**与 JAR 作业相比，应何时使用笔记本作业？**

JAR 作业等效于 spark-submit 作业。 它将执行 JAR，然后你可以查看日志和 Spark UI，以便进行故障排除。 笔记本作业执行指定的笔记本。 你可以在笔记本中导入库，也可以从笔记本调用库。 使用笔记本作业作为 `main` 入口点的优势在于，你可以轻松地在笔记本输出区域中调试生产作业的中间结果。 请参阅 [JAR 作业提示](../jobs.md#jar-jobs)。

**能否连接到我自己的 Hive 元存储？**

可以，Azure Databricks 支持外部 Hive 元存储。 请参阅[外部 Apache Hive 元存储](../data/metastores/external-hive-metastore.md#external-hive-metastore)。