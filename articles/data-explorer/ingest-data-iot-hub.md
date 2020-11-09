---
title: 将数据从 IoT 中心引入到 Azure 数据资源管理器
description: 本文介绍如何将数据从 IoT 中心引入（加载）到 Azure 数据资源管理器中。
author: orspod
ms.author: v-tawe
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: how-to
origin.date: 01/08/2020
ms.date: 09/30/2020
ms.openlocfilehash: 921be4a264ffae2634a153f7552dd290029a8117
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104074"
---
# <a name="ingest-data-from-iot-hub-into-azure-data-explorer"></a>将数据从 IoT 中心引入到 Azure 数据资源管理器 

> [!div class="op_single_selector"]
> * [门户](ingest-data-iot-hub.md)
> * [C#](data-connection-iot-hub-csharp.md)
> * [Python](data-connection-iot-hub-python.md)
> * [Azure Resource Manager 模板](data-connection-iot-hub-resource-manager.md)

[!INCLUDE [data-connector-intro](includes/data-connector-intro.md)]

本文介绍如何将数据从 IoT 中心（大数据流处理平台和 IoT 引入服务）引入 Azure 数据资源管理器。

有关从 IoT 中心将数据引入到 Azure 数据资源管理器的常规信息，请参阅[连接到 IoT 中心](ingest-data-iot-hub-overview.md)。

## <a name="prerequisites"></a>先决条件

* 如果没有 Azure 订阅，请在开始前创建一个[试用 Azure 帐户](https://www.azure.cn/pricing/1rmb-trial/)。
* 创建 [一个测试群集和数据库](create-cluster-database-portal.md)，所用数据库名称为 *testdb* 。
* 介绍如何模拟设备的[示例应用](https://github.com/Azure-Samples/azure-iot-samples-csharp)和文档。
* 用于运行示例应用的 [Visual Studio 2019](https://visualstudio.microsoft.com/vs/)。

## <a name="create-an-iot-hub"></a>创建 IoT 中心

[!INCLUDE [iot-hub-include-create-hub](includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device-to-the-iot-hub"></a>将设备注册到 IoT 中心

[!INCLUDE [iot-hub-get-started-create-device-identity](includes/iot-hub-get-started-create-device-identity.md)]

## <a name="create-a-target-table-in-azure-data-explorer"></a>在 Azure 数据资源管理器中创建目标表

现在，在 Azure 数据资源管理器中创建一个表，IoT 中心会向该表发送数据。 在群集和数据库（已在 [**先决条件**](#prerequisites)中预配）中创建表。

1. 在 Azure 门户中导航到群集，然后选择“查询”。

    ![门户中的 ADX 查询](media/ingest-data-iot-hub/adx-initiate-query.png)

1. 将以下命令复制到窗口中，然后选择“运行”以创建将接收引入数据的表 (TestTable)。

    ```Kusto
    .create table TestTable (temperature: real, humidity: real)
    ```
    
    ![运行创建查询](media/ingest-data-iot-hub/run-create-query.png)

1. 将以下命令复制到窗口中，然后选择“运行”将传入的 JSON 数据映射到表 (TestTable) 的列名和数据类型。

    ```Kusto
    .create table TestTable ingestion json mapping 'TestMapping' '[{"column":"humidity","path":"$.humidity","datatype":"real"},{"column":"temperature","path":"$.temperature","datatype":"real"}]'
    ```

## <a name="connect-azure-data-explorer-table-to-iot-hub"></a>将 Azure 数据资源管理器表连接到 IoT 中心

现在，请通过 Azure 数据资源管理器连接到 IoT 中心。 当此连接完成以后，流入 IoT 中心的数据会流式传输到[创建的目标表](#create-a-target-table-in-azure-data-explorer)。

1. 在工具栏上选择“通知”，以验证 IoT 中心部署是否成功。

1. 在创建的群集下，选择“数据库”，然后选择已创建的数据库“testdb”。
    
    ![选择测试数据库](media/ingest-data-iot-hub/select-database.png)

1. 选择“数据引入”，然后选择“添加数据连接”。

    :::image type="content" source="media/ingest-data-iot-hub/iot-hub-connection.png" alt-text="创建到 IoT 中心的数据连接 - Azure 数据资源管理器":::

### <a name="create-a-data-connection"></a>创建数据连接

1. 使用以下信息填写窗体。 
    
    :::image type="content" source="media/ingest-data-iot-hub/data-connection-pane.png" alt-text="IoT 中心的数据连接窗格 - Azure 数据资源管理器":::

    **设置** | **字段说明**
    |---|---|
    | 数据连接名称 | 要在 Azure 数据资源管理器中创建的连接的名称
    | 订阅 |  事件中心资源所在的订阅 ID。  |
    | IoT 中心 | IoT 中心名称 |
    | 共享访问策略 | 共享访问策略的名称。 必须有读取权限 |
    | 使用者组 |  在 IoT 中心的内置终结点中定义的使用者组 |
    | 事件系统属性 | [IoT 中心事件系统属性](/iot-hub/iot-hub-devguide-messages-construct#system-properties-of-d2c-iot-hub-messages)。 添加系统属性时，[创建](/data-explorer/kusto/management/create-table-command)或[更新](/data-explorer/kusto/management/alter-table-command)表架构和[映射](/data-explorer/kusto/management/mappings)以包括所选属性。 | | | 

#### <a name="target-table"></a>目标表

路由引入数据有两个选项：静态和动态。 本文将使用静态路由，需在其中指定表名、数据格式和映射。 如果事件中心消息包含数据路由信息，则此路由信息将替代默认设置。

1. 填写以下路由设置：
    
    :::image type="content" source="media/ingest-data-iot-hub/default-routing-settings.png" alt-text="默认路由属性 - IoT 中心 - Azure 数据资源管理器":::

     **设置** | **建议的值** | **字段说明**
    |---|---|---|
    | 表名 | *TestTable* | 在“testdb”中创建的表。 |
    | 数据格式 | *JSON* | 支持的格式为 Avro、CSV、JSON、MULTILINE JSON、ORC、PARQUET、PSV、SCSV、SOHSV、TSV、TXT、TSVE、APACHEAVRO 和 W3CLOG。|
    | 映射 | TestMapping | 在 testdb 中创建的[映射](kusto/management/mappings.md)将传入的数据映射到 testdb 的列名称和数据类型 。 对于 JSON、多行 JSON 和 AVRO 是必需的，对于其他格式是可选的。|
    | | |

    > [!WARNING]
    > 如果进行[手动故障转移](/iot-hub/iot-hub-ha-dr#manual-failover)，必须重新创建数据连接。
    
    > [!NOTE]
    > * 无需指定所有默认路由设置。 部分设置也是接受的。
    > * 只有创建数据连接后进入队列的事件才会被引入。

1. 选择“创建”  。

### <a name="event-system-properties-mapping"></a>事件系统属性映射

> [!Note]
> * 单记录事件支持系统属性。
> * 对于 `csv` 映射，属性将添加到记录的开头。 对于 `json` 映射，将根据下拉列表中显示的名称添加属性。

如果在表的“数据源”部分选择了“事件系统属性”，则必须在表架构和映射中包含[系统属性](ingest-data-iot-hub-overview.md#system-properties)。

## <a name="generate-sample-data-for-testing"></a>生成用于测试的示例数据

模拟设备应用程序会连接到 IoT 中心上特定于设备的终结点，并发送模拟的温度和湿度遥测数据。

1. 从 https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip 下载示例 C# 项目并提取 ZIP 存档。

1. 在本地终端窗口中，导航到示例 C# 项目的根文件夹。 然后导航到 **iot-hub\Quickstarts\simulated-device** 文件夹。

1. 在所选文本编辑器中打开 SimulatedDevice.cs 文件。

    将 `s_connectionString` 变量的值替换为[将设备注册到 IoT 中心](#register-a-device-to-the-iot-hub)中的设备连接字符串。 然后将更改保存到 SimulatedDevice.cs 文件。

1. 在本地终端窗口中，运行以下命令以安装模拟设备应用程序所需的包：

    ```cmd/sh
    dotnet restore
    ```

1. 在本地终端窗口中，运行以下命令，生成并运行模拟设备应用程序：

    ```cmd/sh
    dotnet run
    ```

    以下屏幕截图显示了模拟设备应用程序将遥测数据发送到 IoT 中心后的输出：

    ![运行模拟设备](media/ingest-data-iot-hub/simulated-device.png)

## <a name="review-the-data-flow"></a>查看数据流

应用生成数据以后，现在可以看到该数据从 IoT 中心流到群集中的表。

1. 在 Azure 门户中的 IoT 中心下，可以看到应用运行时活动的峰值。

    ![IoT 中心度量值](media/ingest-data-iot-hub/iot-hub-metrics.png)

1. 若要检查到目前为止已向数据库发送的消息数，请在测试数据库中运行以下查询。

    ```Kusto
    TestTable
    | count
    ```

1. 若要查看消息的内容，请运行以下查询：

    ```Kusto
    TestTable
    ```

    结果集：
    
    ![显示引入数据结果](media/ingest-data-iot-hub/show-ingested-data.png)

    > [!NOTE]
    > * Azure 数据资源管理器具有用于数据引入的聚合（批处理）策略，旨在优化引入过程。 默认情况下，该策略配置为 5 分钟或 500 MB 数据，因此你可能会遇到延迟。 有关聚合选项，请参阅[批处理策略](kusto/management/batchingpolicy.md)。 
    > * 配置表以支持流式处理并消除响应时间延迟。 请参阅[流式处理策略](kusto/management/streamingingestionpolicy.md)。 

## <a name="clean-up-resources"></a>清理资源

如果不打算再次使用 IoT 中心，请清理资源组以避免产生费用。

1. 在 Azure 门户的最左侧选择“资源组”，，然后选择创建的资源组。  

    如果左侧菜单处于折叠状态，请选择 ![“展开”按钮](media/ingest-data-event-hub/expand.png) 将其展开。

   ![选择要删除的资源组](media/ingest-data-iot-hub/delete-resources-select.png)

1. 在“test-resource-group”下，选择“删除资源组”。

1. 在新窗口中键入要删除的资源组的名称，然后选择“删除”。

## <a name="next-steps"></a>后续步骤

* [在 Azure 数据资源管理器中查询数据](web-query-data.md)
