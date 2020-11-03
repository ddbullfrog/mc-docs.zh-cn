---
title: 使用 Azure 数据资源管理器 Go SDK 引入数据
description: 本文介绍如何使用 Go SDK 将数据引入（加载）到 Azure 数据资源管理器中。
author: orspod
ms.author: v-tawe
ms.reviewer: abhishgu
ms.service: data-explorer
ms.topic: how-to
origin.date: 08/10/2020
ms.date: 09/30/2020
ms.openlocfilehash: 9058f751ee519ea886063f68b7c34797043401be
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106053"
---
# <a name="ingest-data-using-the-azure-data-explorer-go-sdk"></a>使用 Azure 数据资源管理器 Go SDK 引入数据 

> [!div class="op_single_selector"]
> * [.NET](net-sdk-ingest-data.md)
> * [Python](python-ingest-data.md)
> * [Node](node-ingest-data.md)
> * [Go](go-ingest-data.md)
> * [Java](java-ingest-data.md)

Azure 数据资源管理器是一项快速且高度可缩放的数据探索服务，适用于日志和遥测数据。 它提供了一个用于与 Azure 数据资源管理器服务进行交互的 [Go SDK 客户端库](kusto/api/golang/kusto-golang-client-library.md)。 你可以使用 [Go SDK](https://github.com/Azure/azure-kusto-go) 在 Azure 数据资源管理器群集中引入、控制和查询数据。 

本文首先在测试群集中创建一个表和数据映射。 然后你将使用 Go SDK 将到群集的引入排队并验证结果。

## <a name="prerequisites"></a>先决条件

* 如果没有 Azure 订阅，请在开始前创建一个[试用 Azure 帐户](https://wd.azure.cn/pricing/1rmb-trial/)。
* 安装 [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)。
* 安装满足 [Go SDK 最低要求](kusto/api/golang/kusto-golang-client-library.md#minimum-requirements)的 [Go](https://golang.org/)。 
* 创建 [Azure 数据资源管理器群集和数据库](create-cluster-database-portal.md)。

<!-- * Create an [App Registration and grant it permissions to the database](provision-azure-ad-app.md). Save the client ID and client secret for later use. -->

## <a name="install-the-go-sdk"></a>安装 Go SDK

运行使用 [Go 模块](https://golang.org/ref/mod)的示例应用程序时，会自动安装 Azure 数据资源管理器 Go SDK。 如果已为另一应用程序安装了 Go SDK，请创建一个 Go 模块并提取 Azure 数据资源管理器程序包（使用 `go get`），例如：

```console
go mod init foo.com/bar
go get github.com/Azure/azure-kusto-go/kusto
```

程序包依赖关系将添加到 `go.mod` 文件中。 在你的 Go 应用程序中使用它。

## <a name="review-the-code"></a>查看代码

此[查看代码](#review-the-code)部分是可选的。 如果有兴趣了解代码如何工作，可以查看以下代码片段。 否则，可以跳到[运行应用程序](#run-the-application)。

### <a name="authenticate"></a>Authenticate

在执行任何操作之前，程序都需要向 Azure 数据资源管理器服务进行身份验证。

```go
auth := kusto.Authorization{Config: auth.NewClientCredentialsConfig(clientID, clientSecret, tenantID)}
client, err := kusto.New(kustoEndpoint, auth)
```

将使用服务主体凭据创建 [kusto.Authorization](https://godoc.org/github.com/Azure/azure-kusto-go/kusto#Authorization) 的一个实例。 然后使用该实例以及也接受群集终结点的 [New](https://godoc.org/github.com/Azure/azure-kusto-go/kusto#New]) 函数来创建 [kusto.Client](https://godoc.org/github.com/Azure/azure-kusto-go/kusto#Client)。

### <a name="create-table"></a>创建表

create table 命令是通过一个 [Kusto 语句](https://godoc.org/github.com/Azure/azure-kusto-go/kusto#Stmt)提供的。[Mgmt](https://godoc.org/github.com/Azure/azure-kusto-go/kusto#Client.Mgmt) 函数用来执行管理命令。 它用于执行命令以创建表。 

```go
func createTable(kc *kusto.Client, kustoDB string) {
    _, err := kc.Mgmt(context.Background(), kustoDB, kusto.NewStmt(createTableCommand))
    if err != nil {
        log.Fatal("failed to create table", err)
    }
    log.Printf("Table %s created in DB %s\n", kustoTable, kustoDB)
}
```

> [!TIP]
> 默认情况下 Kusto 语句是常量，可以提高安全性。 [`NewStmt`](https://godoc.org/github.com/Azure/azure-kusto-go/kusto#NewStmt) 接受字符串常量。 [`UnsafeStmt`](https://godoc.org/github.com/Azure/azure-kusto-go/kusto#UnsafeStmt) API 允许使用非常量语句段，但我们不建议使用该 API。

Kusto 的 create table 命令如下所示：

```kusto
.create table StormEvents (StartTime: datetime, EndTime: datetime, EpisodeId: int, EventId: int, State: string, EventType: string, InjuriesDirect: int, InjuriesIndirect: int, DeathsDirect: int, DeathsIndirect: int, DamageProperty: int, DamageCrops: int, Source: string, BeginLocation: string, EndLocation: string, BeginLat: real, BeginLon: real, EndLat: real, EndLon: real, EpisodeNarrative: string, EventNarrative: string, StormSummary: dynamic)
```

### <a name="create-mapping"></a>创建映射

数据映射在引入过程中使用，可将传入数据映射到 Azure 数据资源管理器表中的列。 有关详细信息，请参阅[数据映射](kusto/management/mappings.md)。 映射是按照与表相同的方式创建的，使用 `Mgmt` 函数与数据库名称以及相应的命令。 [示例的 GitHub 存储库](https://github.com/Azure-Samples/Azure-Data-Explorer-Go-SDK-example-to-ingest-data/blob/main/main.go#L20)中提供了完整命令。

```go
func createMapping(kc *kusto.Client, kustoDB string) {
    _, err := kc.Mgmt(context.Background(), kustoDB, kusto.NewStmt(createMappingCommand))
    if err != nil {
        log.Fatal("failed to create mapping - ", err)
    }
    log.Printf("Mapping %s created\n", kustoMappingRefName)
}
```

### <a name="ingest-data"></a>引入数据

引入将使用现有 Azure Blob 存储容器中的文件进行排队。 

```go
func ingestFile(kc *kusto.Client, blobStoreAccountName, blobStoreContainer, blobStoreToken, blobStoreFileName, kustoMappingRefName, kustoDB, kustoTable string) {
    kIngest, err := ingest.New(kc, kustoDB, kustoTable)
    if err != nil {
        log.Fatal("failed to create ingestion client", err)
    }
    blobStorePath := fmt.Sprintf(blobStorePathFormat, blobStoreAccountName, blobStoreContainer, blobStoreFileName, blobStoreToken)
    err = kIngest.FromFile(context.Background(), blobStorePath, ingest.FileFormat(ingest.CSV), ingest.IngestionMappingRef(kustoMappingRefName, ingest.CSV))

    if err != nil {
        log.Fatal("failed to ingest file", err)
    }
    log.Println("Ingested file from -", blobStorePath)
}
```

[引入](https://godoc.org/github.com/Azure/azure-kusto-go/kusto/ingest#Ingestion)客户端是使用 [ingest.New](https://godoc.org/github.com/Azure/azure-kusto-go/kusto/ingest#New) 创建的。 [FromFile](https://godoc.org/github.com/Azure/azure-kusto-go/kusto/ingest#Ingestion.FromFile) 函数用来引用 Azure Blob 存储 URI。 映射引用名称和数据类型是以 [FileOption](https://godoc.org/github.com/Azure/azure-kusto-go/kusto/ingest#FileOption) 形式传递的。 

## <a name="run-the-application"></a>运行此应用程序

1. 从 GitHub 克隆示例代码：

    ```console
    git clone https://github.com/Azure-Samples/Azure-Data-Explorer-Go-SDK-example-to-ingest-data.git
    cd Azure-Data-Explorer-Go-SDK-example-to-ingest-data
    ```

1. 运行示例代码，如 `main.go` 中的以下代码片段所示： 

    ```go
    func main {
        ...
        dropTable(kc, kustoDB)
        createTable(kc, kustoDB)
        createMapping(kc, kustoDB)
        ingestFile(kc, blobStoreAccountName, blobStoreContainer, blobStoreToken, blobStoreFileName, kustoMappingRefName, kustoDB, kustoTable)
        ...
    }
    ```

    > [!TIP]
    > 若要尝试不同的操作组合，可以在 `main.go` 中取消注释/注释相应的函数。

    运行示例代码时，将执行以下操作：
    
    1. **删除表** ：删除 `StormEvents` 表（如果存在）。
    1. **创建表** ：创建 `StormEvents` 表。
    1. **创建映射** ：创建 `StormEvents_CSV_Mapping` 映射。
    1. **引入文件** ：一个 CSV 文件（在 Azure Blob 存储中）将排队等待引入。

1. 若要创建用于身份验证的服务主体，请通过 Azure CLI 使用 [az ad sp create-for-rbac](https://docs.microsoft.com/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) 命令。 采用程序将使用的环境变量的形式设置服务主体信息，包括群集终结点和数据库名称：

    ```console
    export AZURE_SP_CLIENT_ID="<replace with appID>"
    export AZURE_SP_CLIENT_SECRET="<replace with password>"
    export AZURE_SP_TENANT_ID="<replace with tenant>"
    export KUSTO_ENDPOINT="https://<cluster name>.<azure region>.kusto.chinacloudapi.cn"
    export KUSTO_DB="name of the database"
    ```

1. 运行该程序：

    ```console
    go run main.go
    ```

    你将得到类似以下内容的输出：

    ```console
    Connected to Azure Data Explorer
    Using database - testkustodb
    Failed to drop StormEvents table. Maybe it does not exist?
    Table StormEvents created in DB testkustodb
    Mapping StormEvents_CSV_Mapping created
    Ingested file from - https://kustosamplefiles.blob.core.chinacloudapi.cn/samplefiles/StormEvents.csv?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D
    ```

## <a name="validate-and-troubleshoot"></a>验证和故障排除

等待 5 到 10 分钟，以便已排队的引入调度引入进程并将数据加载到 Azure 数据资源管理器中。 

1. 登录到 [https://dataexplorer.azure.cn](https://dataexplorer.azure.cn) 并连接到群集。 然后运行以下命令，以获取 `StormEvents` 表中的记录的计数。

    ```kusto
    StormEvents | count
    ```

2. 在数据库中运行以下命令以查看过去四个小时内是否存在任何失败引入。 在运行之前替换数据库名称。

    ```kusto
    .show ingestion failures
    | where FailedOn > ago(4h) and Database == "<DatabaseName>"
    ```

1. 运行以下命令以查看过去四个小时内所有引入操作的状态。 在运行之前替换数据库名称。

    ```kusto
    .show operations
    | where StartedOn > ago(4h) and Database == "<DatabaseName>" and Operation == "DataIngestPull"
    | summarize arg_max(LastUpdatedOn, *) by OperationId
    ```

## <a name="clean-up-resources"></a>清理资源

如果计划学习我们的其他文章，请保留已创建的资源。 否则，请在数据库中运行以下命令以删除 `StormEvents` 表。

```kusto
.drop table StormEvents
```

## <a name="next-steps"></a>后续步骤

[编写查询](write-queries.md)
