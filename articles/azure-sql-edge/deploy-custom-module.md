---
title: 将 Azure SQL Edge 部署为自定义 IoT Edge 模块
description: 了解如何将 Azure SQL Edge 部署为自定义 IoT Edge 模块
keywords: 部署 SQL Edge 自定义模块
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: v-tawe
ms.reviewer: sstein
origin.date: 10/12/2020
ms.date: 10/13/2020
ms.openlocfilehash: 33ce43dbf8a40aba1c43aef22e261fd495343ce6
ms.sourcegitcommit: 2e443a17dfd1857ecf483fdb84ef087e29c089fe
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/13/2020
ms.locfileid: "91990761"
---
# <a name="deploy-azure-sql-edge-as-custom-iot-edge-module"></a>将 Azure SQL Edge 部署为自定义 IoT Edge 模块 

Azure SQL Edge 是已针对 IoT 和 Azure IoT Edge 部署进行了优化的关系数据库引擎。 它提供了为 IoT 应用和解决方案创建高性能数据存储和处理层的功能。 本快速入门介绍如何将 Azure SQL Edge 部署为自定义 IoT Edge 模块。 若要从 Azure 市场部署 Azure SQL Edge，请参阅[使用门户部署 Azure SQL Edge](deploy-portal.md)。

## <a name="before-you-begin"></a>准备阶段

* 如果没有 Azure 订阅，请创建一个[试用帐户](https://wd.azure.cn/pricing/1rmb-trial/)。
* 登录到 [Azure 门户](https://portal.azure.cn/)。
* 创建 [Azure IoT 中心](../iot-hub/iot-hub-create-through-portal.md)。
* [在 Azure 门户中注册 IoT Edge 设备](../iot-edge/how-to-register-device-portal.md)。
* 准备 IoT Edge 设备，以[在 Azure 门户中部署 IoT Edge 模块](../iot-edge/how-to-deploy-modules-portal.md)。

> [!NOTE]   
> 若要将 Azure Linux VM 部署为 IoT Edge 设备，请参阅这篇[快速入门指南](../iot-edge/quickstart-linux.md)。

## <a name="deploy-sql-edge-as-a-custom-iot-edge-module"></a>将 SQL Edge 部署为自定义 IoT Edge 模块

1. 登录 [Azure 门户](https://portal.azure.cn/)并导航到 IoT 中心。 

2. 在左侧窗格的菜单中选择“IoT Edge”，然后单击要向其部署 SQL Edge 的边缘设备。 

3. 在设备页上，单击“设置模块”。

4. 在“在设备上设置模块:”页的“IoT Edge 模块”下，单击“+ 添加”，然后选择“IoT Edge 模块”  。

5. 在“添加 IoT Edge 模块”边栏选项卡上的“模块设置”下，添加以下详细信息。

    | 名称 | Value |
    |------|-------|
    |IoT Edge 模块名称| AzureSQLEdge |
    |映像 URI| mcr.microsoft.com/azure-sql-edge:latest |
    |重启策略| 通用 |
    |所需状态| “正在运行” |   

    单击“环境变量”

6. 在“添加 IoT Edge 模块”边栏选项卡上的“环境变量”下，添加以下详细信息。

    |**参数**  |**值**|
    |---------|---------|
    | **ACCEPT_EULA** | Y | 
    | PlanId | Azure SQL Edge PlanId，用于标识要部署的 SQL Edge SKU。 可能的值为 asde-developer-on-iot-edge 和 asde-premium-on-iot-edge 。 有关详细信息，请参阅[使用环境变量进行配置](configure.md#configure-by-using-environment-variables) |   
    | **MSSQL_SA_PASSWORD**  | 更改默认值，以便为 SQL Edge 管理员帐户指定强密码。 |  
    | **MSSQL_LCID**   | 更改默认值，以设置要用于 SQL Edge 的所需语言 ID。 例如，1036 为法语。 有关 SQL 语言 ID 的完整列表，请参阅[服务器级排序规则](https://docs.microsoft.com/sql/relational-databases/collations/collation-and-unicode-support#Server-level-collations)|    
    | **MSSQL_COLLATION** | 更改默认值，以设置 SQL Edge 的默认排序规则。 此设置替代语言 ID (LCID) 到排序规则的默认映射。 有关 SQL 排序规则的完整列表，请参阅[服务器级排序规则](https://docs.microsoft.com/sql/relational-databases/collations/collation-and-unicode-support#Server-level-collations)|   

   > [!IMPORTANT]    
   > 请确保正确定义了 `PlanId` 环境变量。 如果未正确定义此值，则 Azure SQL Edge 容器将无法启动。 

    单击“容器创建选项”。

7. 在“添加 IoT Edge 模块”边栏选项卡上的“容器创建选项”下，添加以下详细信息。 请确保根据需要更新以下选项。 

   - 主机端口：将指定主机端口映射到容器中的端口 1433（默认 SQL 端口）。
   - “绑定”和“装载” ：如需部署多个 SQL Edge 模块，请确保更新装载选项，以便为永久性卷新建源和目标对。 若要详细了解装载和卷，请参阅 docker 文档中的[使用卷](https://docs.docker.com/storage/volumes/)。 

    ```json
   {
    "HostConfig": {
        "CapAdd": [
            "SYS_PTRACE"
        ],
        "Binds": [
            "sqlvolume:/sqlvolume"
        ],
        "PortBindings": {
            "1433/tcp": [
                {
                    "HostPort": "1433"
                }
            ]
        },
        "Mounts": [
            {
                "Type": "volume",
                "Source": "sqlvolume",
                "Target": "/var/opt/mssql"
            }
        ]
    },
    "Env": [
        "MSSQL_AGENT_ENABLED=TRUE",
        "ClientTransportType=AMQP_TCP_Only"
    ]
   }
   ```

8. 在“添加 IoT Edge 模块”边栏选项卡上，单击“添加”。

9. 如果需要为部署定义路由，则在“在设备上设置模块”页上，单击“下一步:路由 >”。 否则，单击“审阅 + 创建”。 有关配置路由的详细信息，请参阅[在 IoT Edge 中部署模块和建立路由](../iot-edge/module-composition.md)。

10. 在“在设备上设置模块”页上，单击“创建” 。


## <a name="connect-to-azure-sql-edge"></a>连接到 Azure SQL Edge

下列步骤在容器内部使用 Azure SQL Edge 命令行工具 sqlcmd 来连接 Azure SQL Edge。

> [!NOTE]      
> SQL 命令行工具 (sqlcmd) 在 Azure SQL Edge 容器的 ARM64 版本中不可用。

1. 使用 `docker exec -it` 命令在运行的容器内部启动交互式 Bash Shell。 在下面的示例中，`azuresqledge` 是由 IoT Edge 模块的 `Name` 参数指定的名称。

   ```bash
   sudo docker exec -it azuresqledge "bash"
   ```

2. 在容器内部使用 sqlcmd 进行本地连接。 默认情况下，sqlcmd 不在路径之中，因此需要指定完整路径。

   ```bash
   /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "<YourNewStrong@Passw0rd>"
   ```

   > [!TIP]    
   > 可以省略命令行上提示要输入的密码。

3. 如果成功，应会显示 sqlcmd 命令提示符：`1>`。

## <a name="create-and-query-data"></a>创建和查询数据

以下部分将引导你使用 sqlcmd 和 Transact-SQL 完成新建数据库、添加数据并运行查询的整个过程。

### <a name="create-a-new-database"></a>新建数据库

以下步骤创建一个名为 `TestDB` 的新数据库。

1. 在 sqlcmd 命令提示符中，粘贴以下 Transact-SQL 命令以创建测试数据库：

   ```sql
   CREATE DATABASE TestDB
   Go
   ```

2. 在下一行中，编写一个查询以返回服务器上所有数据库的名称：

   ```sql
   SELECT Name from sys.Databases
   Go
   ```

### <a name="insert-data"></a>插入数据

接下来创建一个新表 `Inventory`，然后插入两个新行。

1. 在 sqlcmd 命令提示符中，将上下文切换到新的 `TestDB` 数据库：

   ```sql
   USE TestDB
   ```

2. 创建名为 `Inventory` 的新表：

   ```sql
   CREATE TABLE Inventory (id INT, name NVARCHAR(50), quantity INT)
   ```

3. 将数据插入新表：

   ```sql
   INSERT INTO Inventory VALUES (1, 'banana', 150); INSERT INTO Inventory VALUES (2, 'orange', 154);
   ```

4. 要执行上述命令的类型 `GO`：

   ```sql
   GO
   ```

### <a name="select-data"></a>选择数据

现在，运行查询以从 `Inventory` 表返回数据。

1. 通过 sqlcmd 命令提示符输入查询，以返回 `Inventory` 表中数量大于 152 的行：

   ```sql
   SELECT * FROM Inventory WHERE quantity > 152;
   ```

2. 执行此命令：

   ```sql
   GO
   ```

### <a name="exit-the-sqlcmd-command-prompt"></a>退出 sqlcmd 命令提示符

1. 要结束 sqlcmd 会话，请键入 `QUIT`：

   ```sql
   QUIT
   ```

2. 要在容器中退出交互式命令提示，请键入 `exit`。 退出交互式 Bash Shell 后，容器将继续运行。

## <a name="connect-from-outside-the-container"></a>从容器外连接

可以从支持 SQL 连接的任何外部 Linux、Windows 或 macOS 工具连接 Azure SQL Edge 实例，并对其运行 SQL 查询。 有关从外部连接到 SQL Edge 容器的详细信息，请参阅[连接和查询 Azure SQL Edge](https://docs.azure.cn/azure-sql-edge/connect)。

在本快速入门中，你在 IoT Edge 设备上部署了 SQL Edge 模块。 

## <a name="next-steps"></a>后续步骤

- [在 SQL Edge 中将机器学习和人工智能与 ONNX 结合使用](onnx-overview.md)
- [使用 IoT Edge 通过 SQL Edge 生成端到端 IoT 解决方案](tutorial-deploy-azure-resources.md)
- [Azure SQL Edge 中的数据流式处理](stream-data.md)
- [排查部署错误](troubleshoot.md)
