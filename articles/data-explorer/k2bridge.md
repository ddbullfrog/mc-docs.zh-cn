---
title: 使用 Kibana 可视化 Azure 数据资源管理器中的数据
description: 本文介绍如何将 Azure 数据资源管理器设置为 Kibana 的数据源
author: orspod
ms.author: v-tawe
ms.reviewer: guregini
ms.service: data-explorer
ms.topic: how-to
origin.date: 03/12/2020
ms.date: 10/30/2020
ms.openlocfilehash: e99fca60c3f820de583607720036d231909e35b0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106532"
---
# <a name="visualize-data-from-azure-data-explorer-in-kibana-with-the-k2bridge-open-source-connector"></a>使用 K2Bridge 开源连接器在 Kibana 中可视化来自 Azure 数据资源管理器的数据

K2Bridge (Kibana-Kusto Bridge) 允许你使用 Azure 数据资源管理器作为数据源，并在 Kibana 中将该数据可视化。 K2Bridge 是一个[开源](https://github.com/microsoft/K2Bridge)的容器化应用程序。 它在 Kibana 实例与 Azure 数据资源管理器群集之间充当代理。 本文介绍了如何使用 K2Bridge 创建该连接。

K2Bridge 将 Kibana 查询转换为 Kusto 查询语言 (KQL)，并将 Azure 数据资源管理器结果发送回 Kibana。

   ![通过 K2Bridge 为 Kibana 建立与 Azure 数据资源管理器的连接](media/k2bridge/k2bridge-chart.png)

K2Bridge 支持 Kibana 的“发现”选项卡，你可以在其中执行以下操作：

* 搜索和浏览数据。
* 筛选结果。
* 在结果网格中添加或删除字段。
* 查看记录内容。
* 保存和共享搜索。

下图显示了一个通过 K2Bridge 绑定到 Azure 数据资源管理器的 Kibana 实例。 Kibana 中的用户体验未更改。

   [![绑定到 Azure 数据资源管理器的 Kibana 页面](media/k2bridge/k2bridge-kibana-page.png)](media/k2bridge/k2bridge-kibana-page.png#lightbox)

## <a name="prerequisites"></a>先决条件

若要在 Kibana 中通过 Azure 数据资源管理器将数据可视化，请准备好下列项：

* [Helm v3](https://github.com/helm/helm#install)，这是 Kubernetes 包管理器。

* Azure Kubernetes 服务 (AKS) 群集或任何其他 Kubernetes 群集。 版本 1.14 到 1.16 已经过测试和验证。 如果你需要一个 AKS 群集，请参阅如何[使用 Azure CLI](/aks/kubernetes-walkthrough) 或[使用 Azure 门户](/aks/kubernetes-walkthrough-portal)部署 AKS 群集。

* 一个 [Azure 数据资源管理器群集](create-cluster-database-portal.md)，包括群集的 URL 和数据库名称。

* 经授权可以在 Azure 数据资源管理器中查看数据的 Azure Active Directory (Azure AD) 服务主体，包括客户端 ID 和客户端密码。

    建议使用具有查看者权限的服务主体，不建议使用较高级别的权限。 [设置群集对 Azure AD 服务主体的查看权限](manage-database-permissions.md#manage-permissions-in-the-azure-portal)。

    有关 Azure AD 服务主体的详细信息，请参阅[创建 Azure AD 服务主体](/active-directory/develop/howto-create-service-principal-portal#create-an-azure-active-directory-application)。

## <a name="run-k2bridge-on-azure-kubernetes-service-aks"></a>在 Azure Kubernetes 服务 (AKS) 上运行 K2Bridge

默认情况下，K2Bridge 的 Helm 图表引用位于 Microsoft 容器注册表 (MCR) 中的公开可用的图像。 MCR 不需要任何凭据。

1. 下载所需的 Helm 图表。

1. 将 Elasticsearch 依赖项添加到 Helm。 此依赖项是必需的，因为 K2Bridge 使用小型内部 Elasticsearch 实例。 与实例服务元数据相关的请求，例如索引模式查询和已保存的查询。 此内部实例不保存任何业务数据。 你可以将该实例视为实现细节。

    1. 若要将 Elasticsearch 依赖项添加到 Helm，请运行以下命令：

        ```bash
        helm repo add elastic https://helm.elastic.co
        helm repo update
        ```

    1. 若要从 GitHub 存储库的图表目录中获取 K2Bridge 图表，请执行以下操作：

        1. 从 [GitHub](https://github.com/microsoft/K2Bridge) 中克隆存储库。
        1. 转到 K2Bridges 根存储库目录。
        1. 运行以下命令：

            ```bash
            helm dependency update charts/k2bridge
            ```

1. 部署 K2Bridge。

    1. 将变量设置为适合你的环境的正确值。

        ```bash
        ADX_URL=[YOUR_ADX_CLUSTER_URL] #For example, https://mycluster.westeurope.kusto.chinacloudapi.cn
        ADX_DATABASE=[YOUR_ADX_DATABASE_NAME]
        ADX_CLIENT_ID=[SERVICE_PRINCIPAL_CLIENT_ID]
        ADX_CLIENT_SECRET=[SERVICE_PRINCIPAL_CLIENT_SECRET]
        ADX_TENANT_ID=[SERVICE_PRINCIPAL_TENANT_ID]
        ```

    2. （可选）启用 Application Insights 遥测。 如果是首次使用 Application Insights，请[创建 Application Insights 资源](/azure-monitor/app/create-new-resource)。 [将检测密钥复制到](/azure-monitor/app/create-new-resource#copy-the-instrumentation-key)一个变量。

        ```bash
        APPLICATION_INSIGHTS_KEY=[INSTRUMENTATION_KEY]
        COLLECT_TELEMETRY=true
        ```

    3. <a name="install-k2bridge-chart"></a> 安装 K2Bridge 图表。

        ```bash
        helm install k2bridge charts/k2bridge -n k2bridge --set image.repository=$REPOSITORY_NAME/$CONTAINER_NAME --set settings.adxClusterUrl="$ADX_URL" --set settings.adxDefaultDatabaseName="$ADX_DATABASE" --set settings.aadClientId="$ADX_CLIENT_ID" --set settings.aadClientSecret="$ADX_CLIENT_SECRET" --set settings.aadTenantId="$ADX_TENANT_ID" [--set image.tag=latest] [--set privateRegistry="$IMAGE_PULL_SECRET_NAME"] [--set settings.collectTelemetry=$COLLECT_TELEMETRY]
        ```

        在 [配置](https://github.com/microsoft/K2Bridge/blob/master/docs/configuration.md)中，可以找到一组完整的配置选项。

    1. <a name="install-kibana-service"></a> 上一个命令的输出建议使用下一个 Helm 命令来部署 Kibana。 （可选）运行以下命令：

        ```bash
        helm install kibana elastic/kibana -n k2bridge --set image=docker.elastic.co/kibana/kibana-oss --set imageTag=6.8.5 --set elasticsearchHosts=http://k2bridge:8080
        ```

    1. 使用端口转发在 localhost 上访问 Kibana。

        ```bash
        kubectl port-forward service/kibana-kibana 5601 --namespace k2bridge
        ```

    2. 转到 http://127.0.0.1:5601 以连接到 Kibana。

    3. 向用户公开 Kibana。 可以使用多种方法执行此操作。 你使用的方法很大程度上取决于你的用例。

        例如，可以将服务公开为负载均衡器服务。 为此，请将 **--set service.type=LoadBalancer** 参数添加到 [早期的 Kibana Helm **install** 命令](#install-kibana-service)。

        然后运行以下命令：

        ```bash
        kubectl get service -w -n k2bridge
        ```

        输出应如下所示：

        ```bash
        NAME            TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
        kibana-kibana   LoadBalancer   xx.xx.xx.xx    <pending>     5601:30128/TCP   4m24s
        ```

        然后，可以使用所显示的已生成的 EXTERNAL-IP 值。 使用该地址通过打开浏览器并转到 \<EXTERNAL-IP\>:5601 来访问 Kibana。

2. 配置索引模式以访问数据。

    在新的 Kibana 实例中：

    1. 打开 Kibana。
    2. 浏览到“管理”。
    3. 选择“索引模式”。
    4. 创建一个索引模式。 索引的名称必须与不带星号 (\*) 的表名或函数名完全匹配。 你可以从列表中复制相关行。

> [!Note]
> 若要在其他 Kubernetes 提供程序上运行 K2Bridge，请更改 values.yaml 中的 Elasticsearch **storageClassName** 值，使其与提供程序建议的值匹配。

## <a name="visualize-data"></a>可视化数据

将 Azure 数据资源管理器配置为 Kibana 的数据源时，可以使用 Kibana 来浏览数据。

1. 在 Kibana 中最左侧的菜单上，选择“发现”选项卡。

1. 在最左边的下拉列表框中，选择一个索引模式。 该模式定义要浏览的数据源。 在本例中，索引模式是一个 Azure 数据资源管理器表。

   ![选择一个索引模式](media/k2bridge/k2bridge-select-an-index-pattern.png)

1. 如果你的数据具有时间筛选器字段，你可以指定时间范围。 在“发现”页的右上方，选择一个时间筛选器。 默认情况下，页面显示过去 15 分钟的数据。

   ![选择一个时间筛选器](media/k2bridge/k2bridge-time-filter.png)

1. 结果表显示前 500 条记录。 你可以展开文档以检查其采用 JSON 格式或表格式的字段数据。

   ![展开的记录](media/k2bridge/k2bridge-expand-record.png)

1. 默认情况下，结果表包含 **_source** 列。 如果时间字段存在，则它还包括 **Time** 列。 你可以通过在最左侧的窗格中选择字段名称旁边的“添加”，将特定列添加到结果表中。

   ![突出显示了“添加”按钮的特定列](media/k2bridge/k2bridge-specific-columns.png)

1. 在查询栏中，可以通过以下方式搜索数据：

    * 输入一个搜索词。
    * 使用 Lucene 查询语法。 例如：
        * 搜索“error”以查找包含此值的所有记录。
        * 搜索“status:200”以获取状态值为 200 的所有记录。
    * 使用逻辑运算符 **AND** 、 **OR** 和 **NOT** 。
    * 使用星号 (\*) 和问号 (?) 通配符。 例如，查询“destination_city:L*”会匹配 destination-city 值以“L”或“l”开头的记录。 （K2Bridge 不区分大小写。）

    ![运行查询](media/k2bridge/k2bridge-run-query.png)

    > [!Tip]
    > 在[搜索](https://github.com/microsoft/K2Bridge/blob/master/docs/searching.md)中，你可以找到更多搜索规则和逻辑。

1. 若要筛选搜索结果，请使用页面最右侧窗格中的字段列表。 可以在字段列表中查看：

    * 字段最靠前的五个值。
    * 包含该字段的记录数。
    * 包含每个值的记录所占的百分比。

    >[!Tip]
    > 使用放大镜可查找具有特定值的所有记录。

    ![突出显示了放大镜的字段列表](media/k2bridge/k2bridge-field-list.png)

    你还可以使用放大镜来筛选结果，并查看结果表中每个记录的结果表格式视图。

     ![突出显示了放大镜的表列表](media/k2bridge/k2bridge-table-list.png)

1. 针对你的搜索选择“保存”或“共享”。

     ![突出显示的按钮，用于保存或共享搜索](media/k2bridge/k2bridge-save-search.png)