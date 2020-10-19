---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/10/2020
title: 最佳做法 - Azure Databricks 上的数据治理 - Azure Databricks
description: 了解数据治理的需求以及可用于在组织中实现这些技术的策略。
ms.openlocfilehash: 0baaccc76e23fbf9f68da6c8fcb04e698f620f85
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937722"
---
# <a name="best-practices-data-governance-on-azure-databricks"></a>最佳做法：Azure Databricks 上的数据管理

本文档介绍数据治理的需求，并分享可用于在组织中实现这些技术的最佳做法和策略。 它演示了一个典型的部署工作流，你可以使用 Azure Databricks 和云原生解决方案来保护和监视从应用程序到存储的每个层。

## <a name="why-is-data-governance-important"></a>为什么数据治理非常重要？

数据治理是一个涵盖性术语，它封装了为安全管理组织内的数据资产而实现的策略和做法。 作为任何成功的数据治理实践的关键原则之一，数据安全性可能是任何大型组织的首要考虑因素。 数据安全性的关键在于，数据团队能够在整个组织中对用户数据访问模式具有出色的可见性和可审核性。 实现有效的数据治理解决方案有助于公司保护其数据免受未经授权的访问，并确保他们制定符合法规要求的规则。

## <a name="governance-challenges"></a>治理挑战

无论你管理的是初创企业还是大型企业的数据，安全团队和平台所有者都会面临一个重大挑战，即确保这些数据是安全的，并根据组织的内部控制进行管理。 世界各地的监管机构正在改变我们对数据的捕获和存储方式的看法。 这些合规性风险只会使本已十分棘手的问题更加复杂。 那么，你如何向那些能够驱动未来用例的用户开放数据呢？ 最终，你应采用数据策略和做法，通过高效应用大量数据存储（即一直在不断增长的存储）来帮助企业实现价值。 当数据团队可以访问许多不同的数据源时，我们就可以获得解决方案来解决世界上最棘手的难题。

考虑云中数据的安全性和可用性时的典型难题：

* 你当前的数据和分析工具是否支持对云中数据的访问控制？ 它们是否提供了对数据在给定工具中移动时所采取措施的可靠日志记录？
* 你现在实施的安全和监视解决方案是否会随着数据湖中数据需求的增长而扩展？ 为少数用户预配和监视数据访问非常容易。 如果要向数百个用户开启数据湖，会发生什么情况？ 数千个呢？
* 是否可以采取任何措施来主动确保数据访问策略得到遵守？ 仅仅监视是不够的，这只是更多的数据。 如果数据可用性仅仅是数据安全的一个挑战，则你应有一个解决方案，以便在整个组织中主动监视和跟踪对这些信息的访问。
* 可以采取哪些步骤来识别现有数据治理解决方案中的不足？

## <a name="how-azure-databricks-addresses-these-challenges"></a>Azure Databricks 如何解决这些难题

* 访问控制：丰富的访问控制套件，一直到存储层。 Azure Databricks 可以通过在平台中使用最先进的 Azure 安全服务来利用其云主干。 在 Spark 群集上启用 Azure Active Directory 凭据传递以控制对数据湖的访问。
* 群集策略：使管理员能够控制对计算资源的访问权限。
* API 优先：使用 Databricks REST API 自动执行预配和权限管理。
* 审核日志：有关跨工作区执行的操作和操作的可靠审核日志已提交到数据湖。 Azure Databricks 可以利用 Azure 的强大功能，跨部署帐户和所配置的任何其他帐户提供数据访问信息。 然后，你可以使用这些信息来启动警报，提示我们潜在的错误行为。

以下部分说明如何使用这些 Azure Databricks 功能来实现治理解决方案。

## <a name="set-up-access-control"></a>设置访问控制

若要设置访问控制，需要保护对存储的访问并实现对单个表的细粒度控制。

### <a name="implement-table-access-control"></a>实现表访问控制

你可以在 Azure Databricks 上[启用表访问控制](../administration-guide/access-control/table-acl.md)，以编程方式授予、拒绝和撤消对来自 Spark SQL API 数据的访问权限。 你可以控制对安全对象（如数据库、表、视图和函数）的访问权限。 假设你的公司有一个用于存储财务数据的数据库。 你可能希望分析师使用这些数据创建财务报表。 但是，数据库中的另一个表中可能存在分析师不应访问的敏感信息。 你可以为用户或组提供从一个表读取数据所需的权限，但拒绝访问第二个表的所有权限。

在下图中，Alice 是一个管理员，拥有财务数据库中的 `shared_data` 和 `private_data` 表。 然后，Alice 为分析师 Oscar 提供了从 `shared_data` 读取的所需权限，但拒绝 `private_data` 的所有权限。

> [!div class="mx-imgBorder"]
> ![授予 select 权限](../_static/images/security/grant-select-statement.png)

Alice 向 Oscar 授予 `SELECT` 权限，以便从 `shared_data` 读取：

> [!div class="mx-imgBorder"]
> ![授予 select 权限表](../_static/images/security/grant-select-table.png)

Alice 拒绝 Oscar 访问 `private_data` 的所有权限：

> [!div class="mx-imgBorder"]
> ![Deny 语句](../_static/images/security/deny-all-statement.png)

你可以通过定义对表的子集的细粒度访问控制或通过设置表的派生视图的权限来进一步实现这一点。

> [!div class="mx-imgBorder"]
> ![Deny 表](../_static/images/security/deny-all-table.png)

## <a name="secure-access-to-azure-data-lake-storage"></a>安全访问 Azure Data Lake Storage

可以通过几种方式从 Azure Databricks 群集访问 Azure Data Lake Storage 中的数据。 此处所述的方法主要与将在相应工作流中使用的数据访问方式对应。 也就是说，你是否会以一种更具交互性的即席方式访问数据，比如开发 ML 模型或构建可操作仪表板？ 在这种情况下，我们建议使用 Azure Active Directory (Azure AD) 凭据传递。 是否要运行需要一次性访问数据湖中的容器的自动化、计划的工作负载？ 那么使用服务主体访问 Azure Data Lake Storage 将是首选。

### <a name="credential-passthrough"></a>凭据直通身份验证

[凭据传递](credential-passthrough/adls-passthrough.md)根据用户的[基于角色的访问控制](/role-based-access-control/overview)为任何预配的文件存储提供用户范围的数据访问控制。 配置群集时，选择并展开“高级选项”以启用凭据传递。 任何用户如果试图访问群集上的数据，都将受到在其相应的文件系统资源上根据其 Active Directory 帐户设置的访问控制约束。

> [!div class="mx-imgBorder"]
> ![群集权限](../_static/images/security/credential-passthrough/azure-credential-passthrough.png)

此解决方案适合于许多交互式用例，并提供了一种简化的方法，要求你只在一个地方管理权限。 通过这种方式，你可以将一个群集分配给多个用户，而无需担心为每个用户预配特定的访问控制。 Azure Databricks 群集上的进程隔离可确保用户凭据不会泄露或以其他方式共享。 此方法还有一个额外好处，可以在 Azure 存储审计日志中记录用户级条目，这有助于平台管理员将存储层操作与特定用户相关联。

此方法存在以下限制：

* 仅支持 Azure Data Lake Storage 文件系统。
* Databricks REST API 访问。
* 表访问控制：Azure Databricks 不建议将凭证直通与表访问控制一起使用。 有关结合使用这两种功能的限制的详细信息，请参阅[限制](credential-passthrough/adls-passthrough.md#limitations)。 有关使用表访问控制的更多信息，请参阅[实现表访问控制](#implement-table-access-control)。
* 不适合长时间运行的作业或查询，因为在用户访问令牌上的生存时间有限。 对于这些类型的工作负载，建议使用[服务主体](#service-principals)来访问数据。

#### <a name="securely-mount-azure-data-lake-storage-using-credential-passthrough"></a>使用凭据传递安全装载 Azure Data Lake Storage

可以将 Azure Data Lake Storage 帐户或其中的文件夹装载到 Databricks 文件系统 (DBFS)，从而提供一种简单而安全的方法来访问数据湖中的数据。 装载是指向数据湖的指针，因此数据永远不会在本地同步。 使用启用了 Azure data Lake Storage 凭据传递的群集装载数据时，对装入点的任何读取或写入操作都将使用 Azure AD 凭据。 此装入点对其他用户可见，但只有具有读写访问权限的用户可以执行以下操作：

* 有权访问基础 Azure Data Lake Storage 存储帐户
* 使用为 Azure Data Lake Storage 凭据传递启用的群集

若要使用凭证传递装载 Azure Data Lake Storage，请按照[使用凭证传递将 Azure Data Lake Storage 装载到 DBFS](credential-passthrough/adls-passthrough.md#aad-passthrough-dbfs) 中的说明操作。

### <a name="service-principals"></a>服务主体

如何授予用户或服务帐户对更长时间运行或更频繁的工作负载的访问权限？ 如果你想利用需要通过 ODBC/JDBC 访问 Azure Databricks 中的表的商业智能工具（如 Power BI 或 Tableau），该怎么办？ 在这些情况下，应使用服务主体和 OAuth。 服务主体是特定 Azure 资源范围内的标识帐户。 在笔记本中生成作业时，可以将以下行添加到作业群集的 Spark 配置或直接在笔记本中运行。 此方法使你可以访问作业范围内的相应文件存储。

```python
spark.conf.set("fs.azure.account.auth.type.<storage-account-name>.dfs.core.chinacloudapi.cn", "OAuth")
spark.conf.set("fs.azure.account.oauth.provider.type.<storage-account-name>.dfs.core.chinacloudapi.cn", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
spark.conf.set("fs.azure.account.oauth2.client.id.<storage-account-name>.dfs.core.chinacloudapi.cn", "<application-id>")
spark.conf.set("fs.azure.account.oauth2.client.secret.<storage-account-name>.dfs.core.chinacloudapi.cn", dbutils.secrets.get(scope = "<scope-name>", key = "<key-name-for-service-credential>"))
spark.conf.set("fs.azure.account.oauth2.client.endpoint.<storage-account-name>.dfs.core.chinacloudapi.cn", "https://login.microsoftonline.com/<directory-id>/oauth2/token")
```

类似地，通过使用服务主体和 OAuth 令牌装载文件存储，可以直接从 Azure Data Lake Storage Gen1 或 Gen2 URI 读取所述数据。 设置上述配置后，现在可以使用 URI 直接访问 Azure Data Lake Storage 中的文件：

```
"abfss://<file-system-name>@<storage-account-name>.dfs.core.chinacloudapi.cn/<directory-name>"
```

群集上以这种方式注册文件系统的所有用户都可以访问文件系统中的数据。

## <a name="manage-cluster-configurations"></a>管理群集配置

[群集策略](../administration-guide/clusters/policies.md)允许 Azure Databricks 管理员定义群集上允许的群集属性，例如实例类型、节点数量、自定义标记等。 当管理员创建策略并将其分配给用户或组时，这些用户只能基于他们有权访问的策略创建群集。 这为管理员提供了对可以创建的群集类型的更高控制力度。

在 JSON 策略定义中定义策略，然后使用[群集策略 UI](../administration-guide/clusters/policies.md#create-a-cluster-policy) 或[群集策略 API](../dev-tools/api/latest/policies.md#cluster-policies-api) 创建群集策略。 仅当用户对至少一个群集策略具有 `create_cluster` 权限或访问权限时，才能创建群集。 扩展你对新分析项目团队的需求，如上所述，管理员现在可以创建群集策略，并将其分配给项目组中的一个或多个用户，这些用户现在可以为团队创建群集，但仅限于群集策略中指定的规则。 下图提供了一个用户的示例，该用户可以访问 `Project Team Cluster Policy` 并根据策略定义创建群集。

> [!div class="mx-imgBorder"]
> ![群集策略](../_static/images/security/cluster-policy-azure.png)

### <a name="automatically-provision-clusters-and-grant-permissions"></a>自动预配群集并授予权限

通过为群集和权限添加终结点，Databricks [REST API 2.0](../dev-tools/api/latest/index.md) 可以轻松地为任何规模的用户和组提供和授予群集资源的权限。 你可以使用[群集 API](../dev-tools/api/latest/clusters.md) 为特定用例创建和配置群集。

然后，可以使用权限 API 对群集应用访问控制。

> [!IMPORTANT]
>
> 权限 API 以[个人预览版](../release-notes/release-types.md)提供。 若要为 Azure Databricks 工作区启用此功能，请与 Azure Databricks 代表联系。

下面的配置示例可能比较适合新的分析项目团队。

要求如下：

* 支持此团队（主要是 SQL 和 Python 用户）的交互式工作负载。
* 使用凭据在对象存储中预配数据源，团队可以通过这些凭据访问与角色绑定的数据。
* 确保用户获取群集资源的相同份额。
* 预配更大的内存优化实例类型。
* 向群集授予权限，以便只有此新项目团队可以访问它。
* 标记此群集，以确保你可以正确地对发生的任何计算成本进行计费。

### <a name="deployment-script"></a>部署脚本

可以使用群集和权限 API 中的 API 终结点来部署此配置。

#### <a name="provision-cluster"></a>设置群集

终结点 - `https:///<databricks-instance>/api/2.0/clusters/create`

```json

{
  "autoscale": {
      "min_workers": 2,
      "max_workers": 20
  },
  "cluster_name": "project team interactive cluster",
  "spark_version": "latest-stable-scala2.11",
  "spark_conf": {
      "spark.Azure Databricks.cluster.profile": "serverless",
      "spark.Azure Databricks.repl.allowedLanguages": "python,sql",
      "spark.Azure Databricks.passthrough.enabled": "true",
      "spark.Azure Databricks.pyspark.enableProcessIsolation": "true"
  },
  "node_type_id": "Standard_D14_v2",
  "ssh_public_keys": [],
  "custom_tags": {
      "ResourceClass": "Serverless",
      "team": "new-project-team"
  },
  "spark_env_vars": {
      "PYSPARK_PYTHON": "/databricks/python3/bin/python3"
  },
  "autotermination_minutes": 60,
  "enable_elastic_disk": true,
  "init_scripts": []
}
```

#### <a name="grant-cluster-permission"></a>授予群集权限

终结点 - `https://<databricks-instance>/api/2.0/permissions/clusters/<cluster_id>`

```json
{
  "access_control_list": [
    {
      "group_name": "project team",
      "permission_level": "CAN_MANAGE"
    }
  ]
}
```

你马上就会拥有一个群集，该群集已经预配对湖中关键数据的安全访问，已锁定除相应团队之外的所有人，已针对退款进行标记，且已配置，满足项目的所有要求。 实现此解决方案还需要在主机云提供商帐户中执行额外的配置步骤，但是也可以自动执行以满足缩放需求。

## <a name="audit-access"></a>审核访问

在 Azure Databricks 中配置访问控制以及控制存储中的数据访问是迈向高效数据治理解决方案的第一步。 但是，完整的解决方案需要审核对数据的访问并提供警报和监视功能。 Databricks 提供一组全面的审核事件来记录 Azure Databricks 用户提供的活动，允许企业监视平台上的详细使用模式。 若要全面了解用户在平台上执行的操作以及访问的数据，应同时使用本机 Azure Databricks 和云提供商审核日志记录功能。

在 Azure Databricks 中配置访问控制以及控制存储帐户中的数据访问是迈向高效数据治理解决方案的第一步。 但是，完整的解决方案还需要能够审核对数据的访问并提供警报和监视功能。 Azure Databricks 提供一组全面的审核事件来记录用户执行的活动，允许企业监视平台上的详细使用模式。

请确保已在 Azure Databricks 中启用了[诊断日志记录](../administration-guide/account-settings/azure-diagnostic-logs.md)。 为帐户启用日志记录后，Azure Databricks 将自动开始向指定的交付位置发送诊断日志。 你还可以选择 `Send to Log Analytics`，这会将诊断数据转发到 Azure Monitor。 下面是一个示例查询，你可以在“日志搜索”框中输入查询已登录到 Azure Databricks 工作区的所有用户及其位置：

> [!div class="mx-imgBorder"]
> ![Azure Monitor](../_static/images/security/azure-monitor.png)

在[几个步骤](/azure-monitor/platform/diagnostic-settings)中，你可以使用 Azure 监视服务或创建实时警报。 Azure 活动日志提供对存储帐户上的操作以及其中的容器所采取的操作的可见性。 也可以在此处配置警报规则。

> [!div class="mx-imgBorder"]
> ![Azure 活动日志](../_static/images/security/azure-activity-log.png)

## <a name="learn-more"></a>了解详细信息

以下是一些资源，可帮助你构建一个全面的数据治理解决方案，以满足组织需求：

* [Databricks 上的访问控制](/administration-guide/access-control/index.html)
* [数据库和表](/data/tables.html)
* [使用机密确保数据安全](/security/secrets/index.html)