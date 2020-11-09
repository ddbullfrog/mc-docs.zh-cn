---
title: 使用 Azure Go SDK 创建 Azure 数据资源管理器群集和数据库
description: 了解如何使用 Azure Go SDK 创建、列出和删除 Azure 数据资源管理器群集和数据库。
author: orspod
ms.author: v-tawe
ms.reviewer: abhishgu
ms.service: data-explorer
ms.topic: how-to
origin.date: 10/28/2020
ms.date: 10/30/2020
ms.openlocfilehash: 67240712693d200e458d2669946267c5864a4c3d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106676"
---
# <a name="create-an-azure-data-explorer-cluster-and-database-using-go"></a>使用 Go 创建 Azure 数据资源管理器群集和数据库

> [!div class="op_single_selector"]
> * [Portal](create-cluster-database-portal.md)
> * [CLI](create-cluster-database-cli.md)
> * [PowerShell](create-cluster-database-powershell.md)
> * [C#](create-cluster-database-csharp.md)
> * [Python](create-cluster-database-python.md)
> * [Go](create-cluster-database-go.md)
> * [ARM 模板](create-cluster-database-resource-manager.md)

Azure 数据资源管理器是一项快速、完全托管的数据分析服务，用于实时分析从应用程序、网站和 IoT 设备等资源流式传输的海量数据。 若要使用 Azure 数据资源管理器，请先创建群集，再在该群集中创建一个或多个数据库。 然后将数据引入（加载）到数据库，以便对其运行查询。 

本文将使用 [Go](https://golang.org/) 创建 Azure 数据资源管理器群集和数据库。 然后，你可以列出和删除新的群集和数据库，并对资源执行操作。

## <a name="prerequisites"></a>先决条件

* 如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://wd.azure.cn/pricing/1rmb-trial/)。
* 安装 [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)。
* 安装合适版本的 Go。 有关支持的版本的详细信息，请参阅 [Azure Go SDK](https://github.com/Azure/azure-sdk-for-go)。

## <a name="review-the-code"></a>查看代码

本部分是可选的。 如果有兴趣了解代码如何工作，可以查看以下代码片段。 否则，可以跳到[运行应用程序](#run-the-application)。

### <a name="authentication"></a>身份验证

在执行任何操作之前，程序都需要向 Azure 数据资源管理器进行身份验证。 [客户端凭据身份验证类型](https://docs.microsoft.com/azure/developer/go/azure-sdk-authorization#use-environment-based-authentication)由 [auth.NewAuthorizerFromEnvironment](https://pkg.go.dev/github.com/Azure/go-autorest/autorest/azure/auth?tab=doc#NewAuthorizerFromEnvironment) 使用，后者会查找以下预定义的环境变量：`AZURE_CLIENT_ID`、`AZURE_CLIENT_SECRET`、`AZURE_TENANT_ID`。

以下示例展示了如何使用此方法创建 [kusto.ClustersClient](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go@v0.0.0-20200513030755-ac906323d9fe/services/kusto/mgmt/2020-02-15/kusto?tab=doc#ClustersClient)：

```go
func getClustersClient(subscription string) kusto.ClustersClient {
    client := kusto.NewClustersClient(subscription)
    authR, err := auth.NewAuthorizerFromEnvironment()
    if err != nil {
        log.Fatal(err)
    }
    client.Authorizer = authR

    return client
}
```

> [!TIP]
> 如果你为身份验证安装并配置了 Azure CLI，请使用 [auth.NewAuthorizerFromCLIWithResource](https://pkg.go.dev/github.com/Azure/go-autorest/autorest/azure/auth?tab=doc#NewAuthorizerFromCLIWithResource) 函数进行本地开发。

### <a name="create-cluster"></a>创建群集

对 `kusto.ClustersClient` 使用 [CreateOrUpdate](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go@v0.0.0-20200513030755-ac906323d9fe/services/kusto/mgmt/2020-02-15/kusto?tab=doc#ClustersClient.CreateOrUpdate) 函数来创建新的 Azure 数据资源管理器群集。 等待该过程完成，然后检查结果。

```go
func createCluster(sub, name, location, rgName string) {
    ...
    result, err := client.CreateOrUpdate(ctx, rgName, name, kusto.Cluster{Location: &location, Sku: &kusto.AzureSku{Name: kusto.DevNoSLAStandardD11V2, Capacity: &numInstances, Tier: kusto.Basic}})
    ...
    err = result.WaitForCompletionRef(context.Background(), client.Client)
    ...
    r, err := result.Result(client)
}
```

### <a name="list-clusters"></a>列出群集

对 `kusto.ClustersClient` 使用 [ListByResourceGroup](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go@v0.0.0-20200513030755-ac906323d9fe/services/kusto/mgmt/2020-02-15/kusto?tab=doc#ClustersClient.ListByResourceGroup) 函数来获取 [kusto.ClusterListResult](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go@v0.0.0-20200513030755-ac906323d9fe/services/kusto/mgmt/2020-02-15/kusto?tab=doc#ClusterListResult)，后者将进行迭代以便以表格式显示输出。


```go
func listClusters(sub, rgName string) {
    ...
    result, err := getClustersClient(sub).ListByResourceGroup(ctx, rgName)
    ...
    for _, c := range *result.Value {
        // setup tabular representation
    }
    ...
}
```

### <a name="create-database"></a>创建数据库

对 [kusto.DatabasesClient](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go@v0.0.0-20200513030755-ac906323d9fe/services/kusto/mgmt/2020-02-15/kusto?tab=doc#DatabasesClient) 使用 [CreateOrUpdate](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go@v0.0.0-20200513030755-ac906323d9fe/services/kusto/mgmt/2020-02-15/kusto?tab=doc#DatabasesClient.CreateOrUpdate) 函数，以在现有群集中创建新的 Azure 数据资源管理器数据库。 等待该过程完成，然后检查结果。


```go
func createDatabase(sub, rgName, clusterName, location, dbName string) {
    future, err := client.CreateOrUpdate(ctx, rgName, clusterName, dbName, kusto.ReadWriteDatabase{Kind: kusto.KindReadWrite, Location: &location})
    ...
    err = future.WaitForCompletionRef(context.Background(), client.Client)
    ...
    r, err := future.Result(client)
    ...
}
```

### <a name="list-databases"></a>列出数据库

对 `kusto.DatabasesClient` 使用 [ListByCluster](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go@v0.0.0-20200513030755-ac906323d9fe/services/kusto/mgmt/2020-02-15/kusto?tab=doc#DatabasesClient.ListByCluster) 函数以获取 [kusto.DatabaseListResult](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go@v0.0.0-20200513030755-ac906323d9fe/services/kusto/mgmt/2020-02-15/kusto?tab=doc#DatabaseListResult)，后者将进行迭代以便以表格格式显示输出。


```go
func listDatabases(sub, rgName, clusterName string) {
    result, err := getDBClient(sub).ListByCluster(ctx, rgName, clusterName)
    ...
    for _, db := range *result.Value {
        // setup tabular representation
    }
    ...
}
```

### <a name="delete-database"></a>删除数据库

对 `kusto.DatabasesClient` 使用 [Delete](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go@v0.0.0-20200513030755-ac906323d9fe/services/kusto/mgmt/2020-02-15/kusto?tab=doc#DatabasesClient.Delete) 函数以删除群集中的现有数据库。 等待该过程完成，然后检查结果。

```go
func deleteDatabase(sub, rgName, clusterName, dbName string) {
    ...
    future, err := getDBClient(sub).Delete(ctx, rgName, clusterName, dbName)
    ...
    err = future.WaitForCompletionRef(context.Background(), client.Client)
    ...
    r, err := future.Result(client)
    if r.StatusCode == 200 {
        // determine success or failure
    }
    ...
}
```

### <a name="delete-cluster"></a>删除群集

对 `kusto.ClustersClient` 使用 [Delete](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go@v0.0.0-20200513030755-ac906323d9fe/services/kusto/mgmt/2020-02-15/kusto?tab=doc#ClustersClient.Delete) 函数以删除群集。 等待该过程完成，然后检查结果。

```go
func deleteCluster(sub, clusterName, rgName string) {
    result, err := client.Delete(ctx, rgName, clusterName)
    ...
    err = result.WaitForCompletionRef(context.Background(), client.Client)
    ...
    r, err := result.Result(client)
    if r.StatusCode == 200 {
        // determine success or failure
    }
    ...
}
```

## <a name="run-the-application"></a>运行此应用程序

按原样运行示例代码时，将执行以下操作：
    
1. 创建一个 Azure 数据资源管理器群集。
1. 列出指定资源组中的所有 Azure 数据资源管理器群集。
1. 在现有群集中创建一个 Azure 数据资源管理器数据库。
1. 列出指定群集中的所有数据库。
1. 删除数据库。
1. 删除群集。


    ```go
    func main() {
        createCluster(subscription, clusterNamePrefix+clusterName, location, rgName)
        listClusters(subscription, rgName)
        createDatabase(subscription, rgName, clusterNamePrefix+clusterName, location, dbNamePrefix+databaseName)
        listDatabases(subscription, rgName, clusterNamePrefix+clusterName)
        deleteDatabase(subscription, rgName, clusterNamePrefix+clusterName, dbNamePrefix+databaseName)
        deleteCluster(subscription, clusterNamePrefix+clusterName, rgName)
    }
    ```

    > [!TIP]
    > 若要尝试不同的操作组合，可以根据需要在 `main.go` 中取消注释和注释相应的函数。

1. 从 GitHub 克隆示例代码：

    ```console
    git clone https://github.com/Azure-Samples/azure-data-explorer-go-cluster-management.git
    cd azure-data-explorer-go-cluster-management
    ```

2. 程序使用客户端凭据进行身份验证。 使用 Azure CLI [az ad sp create-for-rbac](/cli/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) 命令创建服务主体。 保存客户端 ID、客户端密码和租户 ID 信息，以便在下一步中使用。

3. 导出所需的环境变量，包括服务主体信息。 输入要在其中创建群集的订阅 ID、资源组和区域。

    ```console
    export AZURE_CLIENT_ID="<enter service principal client ID>"
    export AZURE_CLIENT_SECRET="<enter service principal client secret>"
    export AZURE_TENANT_ID="<enter tenant ID>"

    export SUBSCRIPTION="<enter subscription ID>"
    export RESOURCE_GROUP="<enter resource group name>"
    export LOCATION="<enter azure location e.g. Southeast Asia>"

    export CLUSTER_NAME_PREFIX="<enter prefix (cluster name will be [prefix]-ADXTestCluster)>"
    export DATABASE_NAME_PREFIX="<enter prefix (database name will be [prefix]-ADXTestDB)>"
    ```
    
    > [!TIP]
    > 在生产方案中，你可能会使用环境变量向应用程序提供凭据。 如[查看代码](#review-the-code)中所述，对于本地开发，如果你为身份验证安装并配置了 Azure CLI，请使用 [auth.NewAuthorizerFromCLIWithResource](https://pkg.go.dev/github.com/Azure/go-autorest/autorest/azure/auth?tab=doc#NewAuthorizerFromCLIWithResource)。 在这种情况下，不需要创建服务主体。


4. 运行该程序：

    ```console
    go run main.go
    ```

    你会看到类似于以下内容的输出：

    ```console
    waiting for cluster creation to complete - fooADXTestCluster
    created cluster fooADXTestCluster
    listing clusters in resource group <your resource group>
    +-------------------+---------+----------------+-----------+-------------------------------------------------------------+
    |       NAME        |  STATE  |    LOCATION    | INSTANCES |                            URI                              |
    +-------------------+---------+----------------+-----------+-------------------------------------------------------------+
    | fooADXTestCluster | Running | China East 2   |         1 | https://fooADXTestCluster.chinaeast2.kusto.chinacloudapi.cn |
    +-------------------+---------+----------------+-----------+-------------------------------------------------------------+
    
    waiting for database creation to complete - barADXTestDB
    created DB fooADXTestCluster/barADXTestDB with ID /subscriptions/<your subscription ID>/resourceGroups/<your resource group>/providers/Microsoft.Kusto/Clusters/fooADXTestCluster/Databases/barADXTestDB and type Microsoft.Kusto/Clusters/Databases
    
    listing databases in cluster fooADXTestCluster
    +--------------------------------+-----------+----------------+------------------------------------+
    |              NAME              |   STATE   |    LOCATION    |                TYPE                |
    +--------------------------------+-----------+----------------+------------------------------------+
    | fooADXTestCluster/barADXTestDB | Succeeded |  China East 2  | Microsoft.Kusto/Clusters/Databases |
    +--------------------------------+-----------+----------------+------------------------------------+
    
    waiting for database deletion to complete - barADXTestDB
    deleted DB barADXTestDB from cluster fooADXTestCluster

    waiting for cluster deletion to complete - fooADXTestCluster
    deleted ADX cluster fooADXTestCluster from resource group <your resource group>
    ```

## <a name="clean-up-resources"></a>清理资源

如果未使用本文中的示例代码以编程方式删除群集，请使用 [Azure CLI](create-cluster-database-cli.md#clean-up-resources) 手动删除它们。

## <a name="next-steps"></a>后续步骤

[使用 Azure 数据资源管理器 Go SDK 引入数据](go-ingest-data.md)
