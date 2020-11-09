---
title: Azure 数据资源管理器工具和集成概述 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的工具和集成。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: olgolden
ms.service: data-explorer
ms.topic: conceptual
origin.date: 07/08/2020
ms.date: 09/30/2020
ms.openlocfilehash: e7fad9ffaedb0cecf0be5b798943d675bf81dc37
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105347"
---
# <a name="azure-data-explorer-tools-and-integrations-overview"></a>Azure 数据资源管理器工具和集成概述

Azure 数据资源管理器是一项快速、完全托管的数据分析服务，用于实时分析从应用程序、网站、IoT 设备及其他资源流式传输的大量数据。 Azure 数据资源管理器收集、存储和分析多种不同数据，以改良产品、增强客户体验、监视设备和改进操作。 

Azure 数据资源管理器提供不同的工具和集成，以实现数据引入、查询、可视化、编排及其他功能。 用户除了可以利用 Azure 数据资源管理器的本机服务以外，还可通过它与各种不同产品和平台轻松集成，启用各种不同的客户用例，以及简化工作流并降低成本来优化业务流程。 

本文提供了 Azure 数据资源管理器工具、连接器和集成的列表，其中包含相关文档的链接以便你获取更多信息。

## <a name="ingest-data"></a>引入数据 

数据引入是用于从一个或多个源将数据记录加载到 Azure 数据资源管理器中的进程。 引入后，数据即可用于查询。 Azure 数据资源管理器为数据引入提供了多种工具和连接器。 

### <a name="azure-data-explorer-ingestion-tools"></a>Azure 数据资源管理器引入工具

* [LightIngest](lightingest.md) - 一种帮助实用程序，可以将数据临时引入 Azure 数据资源管理器

<!--
* One-click Ingestion: [overview](ingest-data-one-click.md) and ingest data [from a container to a new table](one-click-ingestion-new-table.md)
or [from a local file to an existing table](one-click-ingestion-existing-table.md)
-->

### <a name="ingestion-integrations"></a>引入集成

<!-- * Event Grid: [Ingest from Event Grid overview](ingest-data-event-grid-overview.md) and using the [Azure portal](ingest-data-event-grid.md), [C#](data-connection-event-grid-csharp.md), [Python](data-connection-event-grid-python.md) or [Azure Resource Manager template](data-connection-event-grid-resource-manager.md) -->
<!-- * [Azure Synapse Apache Spark](https://docs.microsoft.com/azure/synapse-analytics/quickstart-connect-azure-data-explorer?context=/azure/data-explorer/context/context) -->

* 事件中心：[从事件中心引入概述](ingest-data-event-hub-overview.md)以及使用 [Azure 门户](ingest-data-event-hub.md)、[C#](data-connection-event-hub-csharp.md)、[Python](data-connection-event-hub-python.md) 或 [Azure 资源管理器模板](data-connection-event-hub-resource-manager.md)
* IoT 中心：[从 IoT 中心引入概述](ingest-data-iot-hub-overview.md)以及使用 [Azure 门户](ingest-data-iot-hub.md)、[C#](data-connection-iot-hub-csharp.md)、[Python](data-connection-iot-hub-python.md) 或 [Azure 资源管理器模板](data-connection-iot-hub-resource-manager.md)
* [Logstash](ingest-data-logstash.md)
* Azure 数据工厂：[集成概述](data-factory-integration.md)、[复制数据](data-factory-load-data.md)、[使用 Azure 数据工厂模板批量复制](data-factory-template.md)和[使用 Azure 数据工厂命令活动运行 Azure 数据资源管理器控制命令](data-factory-command-activity.md)
* Apache Spark[](spark-connector.md)
* [Apache Kafka](ingest-data-kafka.md)
* [Cosmos DB](https://github.com/Azure/azure-kusto-labs/tree/master/cosmosdb-adx-integration)

<!-- * [Power Automate](flow.md) -->

## <a name="query-data"></a>查询数据

### <a name="azure-data-explorer-query-tools"></a>Azure 数据资源管理器查询工具

Azure 数据资源管理器中有几种工具可用于运行查询。

* Kusto.Explorer
    * [安装和用户界面](kusto/tools/kusto-explorer.md)、[使用 Kusto.Explorer](kusto/tools/kusto-explorer-using.md)
    * 其他主题包括[选项](kusto/tools/kusto-explorer-options.md)、[故障排除](kusto/tools/kusto-explorer-troubleshooting.md)、[键盘快捷方式](kusto/tools/kusto-explorer-shortcuts.md)、[代码重构](kusto/tools/kusto-explorer-refactor.md)、[代码导航](kusto/tools/kusto-explorer-codenav.md)和[代码分析](kusto/tools/kusto-explorer-code-analyzer.md)
* [Web UI](web-query-data.md)
* [Kusto.Cli](kusto/tools/kusto-cli.md)

### <a name="query-integrations"></a>查询集成

* [Azure Monitor](query-monitor-data.md)
* [Azure 数据湖](data-lake-query-data.md)
* Apache Spark
* Azure Data Studio：[Kusto 扩展概述](https://docs.microsoft.com/sql/azure-data-studio/extensions/kusto-extension?context=%252fazure%252fdata-explorer%252fcontext%252fcontext)、[使用 Kusto](https://docs.microsoft.com/sql/azure-data-studio/notebooks/notebooks-kusto-kernel?context=%252fazure%252fdata-explorer%252fcontext%252fcontext) 和[使用 Kqlmagic](https://docs.microsoft.com/sql/azure-data-studio/notebooks-kqlmagic?context=%252fazure%252fdata-explorer%252fcontext%252fcontext)

## <a name="visualizations-dashboards-and-reporting"></a>可视化、仪表板和报告

[可视化概述](viz-overview.md)详细介绍了数据可视化、仪表板和报告选项。 

## <a name="notebook-connectivity"></a>Notebook 连接

* [Jupyter Notebook](kqlmagic.md)
* Azure Data Studio：[Kusto 扩展概述](https://docs.microsoft.com/sql/azure-data-studio/extensions/kusto-extension?context=%252fazure%252fdata-explorer%252fcontext%252fcontext)、[使用 Kusto](https://docs.microsoft.com/sql/azure-data-studio/notebooks/notebooks-kusto-kernel?context=%252fazure%252fdata-explorer%252fcontext%252fcontext) 和[使用 Kqlmagic](https://docs.microsoft.com/sql/azure-data-studio/notebooks-kqlmagic?context=%252fazure%252fdata-explorer%252fcontext%252fcontext)

<!-- ## Orchestration -->

<!-- * Power Automate
    * [Power Automate connector](flow.md)
    * [Power Automate connector usage examples](flow-usage.md) -->
<!-- * [Microsoft Logic App](kusto/tools/logicapps.md)  -->
<!-- * [Azure Data Factory](data-factory-integration.md) -->

<!-- ## Share data

* [Azure Data Share](data-share.md) -->

<!-- ## Source control integration

* [Azure Pipelines](devops.md) 
* [Sync Kusto](kusto/tools/synckusto.md)  -->

<!--Open Source Tools-->
