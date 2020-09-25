---
title: 转换数据
description: 在 Azure 数据工厂中利用 Hadoop、机器学习或 Azure Data Lake Analytics 转换或处理数据。
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
author: WenJason
ms.author: v-jay
manager: digimobile
ms.custom: seo-lt-2019
origin.date: 07/31/2018
ms.date: 09/21/2020
ms.openlocfilehash: 1a3fd39e040d93f56797a19e873acfd27e5dfba9
ms.sourcegitcommit: f5d53d42d58c76bb41da4ea1ff71e204e92ab1a7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90523739"
---
# <a name="transform-data-in-azure-data-factory"></a>在 Azure 数据工厂中转换数据

> [!div class="op_single_selector"]
> * [Hive](transform-data-using-hadoop-hive.md)  
> * [Pig](transform-data-using-hadoop-pig.md)  
> * [MapReduce](transform-data-using-hadoop-map-reduce.md)  
> * [HDInsight Streaming](transform-data-using-hadoop-streaming.md)
> * [HDInsight Spark](transform-data-using-spark.md)
> * [存储过程](transform-data-using-stored-procedure.md)
> * [.NET 自定义](transform-data-using-dotnet-custom-activity.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

## <a name="overview"></a>概述
本文介绍了 Azure 数据工厂中的数据转换活动，可利用这些活动将原始数据转换和处理为大规模预测和见解。 转换活动在计算环境（例如 Azure HDInsight）中执行。 其提供了相关文章链接，内附各转换活动的详细信息。

数据工厂支持以下数据转换活动，这些活动可单独添加到[管道](concepts-pipelines-activities.md)，还可与其他活动关联在一起。

## <a name="external-transformations"></a>外部转换

可以手动编写代码转换并自行管理外部计算环境。

### <a name="hdinsight-hive-activity"></a>HDInsight Hive 活动
数据工厂管道中的 HDInsight Hive 活动会在自己的或基于 Windows/Linux 的按需 HDInsight 群集上执行 Hive 查询。 有关此活动的详细信息，请参阅 [Hive 活动](transform-data-using-hadoop-hive.md)一文。 

### <a name="hdinsight-pig-activity"></a>HDInsight Pig 活动
数据工厂管道中的 HDInsight Pig 活动会在自己或基于 Windows/Linux 的按需 HDInsight 群集上执行 Pig 查询。 有关此活动的详细信息，请参阅 [Pig 活动](transform-data-using-hadoop-pig.md)一文。 

### <a name="hdinsight-mapreduce-activity"></a>HDInsight MapReduce 活动
数据工厂管道中的 HDInsight MapReduce 活动会在自己或基于 Windows/Linux 的按需 HDInsight 群集上执行 MapReduce 程序。 有关此活动的详细信息，请参阅 [MapReduce 活动](transform-data-using-hadoop-map-reduce.md)一文。

### <a name="hdinsight-streaming-activity"></a>HDInsight Streaming 活动
数据工厂管道中的 HDInsight 流式处理活动会在自己的或基于 Windows/Linux 的按需 HDInsight 群集上执行 Hadoop 流式处理程序。 有关此活动的详细信息，请参阅 [HDInsight Streaming 活动](transform-data-using-hadoop-streaming.md)。

### <a name="hdinsight-spark-activity"></a>HDInsight Spark 活动
数据工厂管道中的 HDInsight Spark 活动在自己的 HDInsight 群集上执行 Spark 程序。 有关详细信息，请参阅[从 Azure 数据工厂调用 Spark 程序](transform-data-using-spark.md)。 

### <a name="stored-procedure-activity"></a>存储过程活动
可使用数据工厂管道中的 SQL Server 存储过程活动调用以下数据存储之一中的存储过程：你的企业或 Azure VM 中的 Azure SQL 数据库、Azure Synapse Analytics（以前称为 SQL 数据仓库）、SQL Server 数据库。 有关详细信息，请参阅[存储过程活动](transform-data-using-stored-procedure.md)一文。  

### <a name="custom-activity"></a>自定义活动
如果需要采用数据工厂不支持的方式转换数据，可以使用自己的数据处理逻辑创建自定义活动，并在管道中使用该活动。 可以使用 Azure Batch 服务或 Azure HDInsight 群集配置要运行的自定义 .NET 活动。 有关详细信息，请参阅[使用自定义活动](transform-data-using-dotnet-custom-activity.md)文章。 

可以创建一项自定义活动，在安装了 R 的 HDInsight 群集上运行 R 脚本。 请参阅 [Run R Script using Azure Data Factory](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/RunRScriptUsingADFSample)（使用 Azure 数据工厂运行 R 脚本）。 

### <a name="compute-environments"></a>计算环境
为计算环境创建链接服务，并在定义转换活动时使用该服务。 数据工厂支持两类计算环境。 

- **按需**：在这种情况下，计算环境由数据工厂完全托管。 作业提交到进程数据前，数据工厂服务会自动创建计算环境，作业完成后则自动将其删除。 针对作业执行、群集管理和启动操作，可以配置和控制按需计算环境的粒度设置。 
- **自带**：在这种情况下，可将自己的计算环境（例如 HDInsight 群集）注册为数据工厂中的链接服务。 计算环境由用户进行管理，数据工厂服务使用它执行活动。 

有关数据工厂支持的计算服务列表，请参阅 [计算链接服务](compute-linked-services.md)文章。 

## <a name="next-steps"></a>后续步骤
请参阅以下使用转换活动的示例教程：[教程：使用 Spark 转换数据](tutorial-transform-data-spark-powershell.md)
