---
title: 使用 ID 代理（预览版）进行凭据管理 - Azure HDInsight
description: 了解 HDInsight ID 代理如何简化已加入域的 Apache Hadoop 群集的身份验证。
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.topic: conceptual
ms.date: 12/12/2019
ms.openlocfilehash: d88ced955b96fee3a4ea4235356096cb55f36c25
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937204"
---
# <a name="azure-hdinsight-id-broker-preview"></a>Azure HDInsight ID 代理（预览版）

本文介绍了如何在 Azure HDInsight 中设置和使用 HDInsight ID 代理 (HIB) 功能。 可以使用此功能获得对 Apache Ambari 的新式 OAuth 身份验证，同时执行多重身份验证 (MFA)，而无需在 Azure Active Directory 域服务 (AAD-DS) 中使用旧密码哈希。

## <a name="overview"></a>概述

在以下情况下，HIB 简化了复杂的身份验证设置：

* 你的组织依赖于联合身份验证对访问云资源的用户进行身份验证。 以前，若要使用 HDInsight 企业安全性套餐 (ESP) 群集，你必须启用从本地环境到 Azure Active Directory (Azure AD) 的密码哈希同步。 对于某些组织而言，此要求可能比较困难或不理想。

* 你的组织希望对基于 Web / HTTP 的 Apache Ambari 和其他群集资源的访问实施 MFA。

HIB 提供身份验证基础结构，支持协议从 OAuth（新式）转换到 Kerberos（旧式），而无需将密码哈希同步到 AAD-DS。 此基础结构由 Windows Server VM（ID 代理节点）上运行的组件以及群集网关节点组成。

下图显示了启用 ID 代理后针对所有用户（包括联合用户）的基于 OAuth 的新式身份验证流：

![采用 ID 代理的身份验证流](./media/identity-broker/identity-broker-architecture.png)

在此图中，客户端（即浏览器或应用）需要先获取 OAuth 令牌，然后在 HTTP 请求中将令牌提供给网关。 如果已登录到其他 Azure 服务（例如 Azure 门户），可以使用单一登录 (SSO) 体验登录到 HDInsight 群集。

仍有许多旧版应用程序仅支持基本身份验证（即用户名/密码）。 对于这些情况，仍然可以使用 HTTP 基本身份验证连接到群集网关。 在此设置中，必须确保从网关节点到联合终结点（ADFS 终结点）的网络连接性，从而确保网关节点的直接视线。

使用下表根据你的组织需求确定最佳身份验证选项：

|身份验证选项 |HDInsight 配置 | 需考虑的因素 |
|---|---|---|
| 完全 OAuth | ESP + HIB | 1.最安全选项（支持 MFA）2.    不需要哈希传递同步。 3.  不支持本地帐户的 ssh/kinit/keytab 访问，这些帐户在 AAD-DS 中没有密码哈希。 4.   仅限云的帐户仍然可以 ssh/kinit/keytab。 5. 通过 Oauth 6 实现的对 Ambari 的基于 Web 的访问。  需要更新旧版应用（JDBC/ODBC 等）以支持 OAuth。|
| OAuth + 基本身份验证 | ESP + HIB | 1.通过 Oauth 2 实现的对 Ambari 的基于 Web 的访问。 旧版应用继续使用基本身份验证。3. 必须禁用 MFA 才能进行基本身份验证访问。 4. 不需要哈希传递同步。 5. 不支持本地帐户的 ssh/kinit/keytab 访问，这些帐户在 AAD-DS 中没有密码哈希。 6. 仅限云的帐户仍然可以 ssh/kinit。 |
| 完全基本身份验证 | ESP | 1.最类似于本地设置。 2. 需要将密码哈希同步到 AAD-DS。 3. 本地帐户可以 ssh / kinit 或使用 keytab。 4. 如果后备存储为 ADLS Gen2，则必须禁用 MFA |


## <a name="enable-hdinsight-id-broker"></a>启用 HDInsight ID 代理

若要创建启用了 ID 代理的 ESP 群集，请执行以下步骤：

1. 登录到 [Azure 门户](https://portal.azure.cn)。
1. 执行 ESP 群集的基本创建步骤。 有关详细信息，请参阅[创建使用 ESP 的 HDInsight 群集](apache-domain-joined-configure-using-azure-adds.md#create-an-hdinsight-cluster-with-esp)。
1. 选择“启用 HDInsight ID 代理”。

ID 代理功能将向群集添加一个额外的 VM。 此 VM 是 ID 代理节点，包括了用来支持身份验证的服务器组件。 ID 代理节点以域加入方式加入到 Azure AD DS 域。

![用于启用 ID 代理的选项](./media/identity-broker/identity-broker-enable.png)

### <a name="using-azure-resource-manager-templates"></a>使用 Azure 资源管理器模板
如果将名为 `idbrokernode` 且具有以下特性的一个新角色添加到模板的计算配置文件中，则会在启用 ID 代理节点的情况下创建集群：

```json
.
.
.
"computeProfile": {
    "roles": [
        {
            "autoscale": null,
            "name": "headnode",
           ....
        },
        {
            "autoscale": null,
            "name": "workernode",
            ....
        },
        {
            "autoscale": null,
            "name": "idbrokernode",
            "targetInstanceCount": 1,
            "hardwareProfile": {
                "vmSize": "Standard_A2_V2"
            },
            "virtualNetworkProfile": {
                "id": "string",
                "subnet": "string"
            },
            "scriptActions": [],
            "dataDisksGroups": null
        }
    ]
}
.
.
.
```

## <a name="tool-integration"></a>工具集成

HDIsngith 工具已更新为原生支持 OAuth。 我们强烈建议使用这些工具对群集进行基于新式 OAuth 的访问。 HDInsight [IntelliJ 插件](/hdinsight/spark/apache-spark-intellij-tool-plugin#integrate-with-hdinsight-identity-broker-hib)可以用于基于 Java 的应用程序，例如 Scala。 [适用于 VS Code 的 Spark＆Hive 工具](/hdinsight/hdinsight-for-vscode)可用于 PySpark 和 Hive 作业。 它们支持批处理和交互式作业。

## <a name="ssh-access-without-a-password-hash-in-azure-ad-ds"></a>在 Azure AD DS 中在没有密码哈希的情况下进行 SSH 访问

|SSH 选项 |需考虑的因素 |
|---|---|
| 本地 VM 帐户（例如 sshuser） | 1.你创建群集时提供了此帐户。 2.  此帐户没有 kerberos 身份验证 |
| 仅限云的帐户（例如 alice@contoso.onmicrosoft.com） | 1.密码哈希在 AAD-DS 2 中可用。 Kerberos 身份验证可以通过 SSH kerberos 进行 |
| 本地帐户（例如 alice@contoso.com） | 1.仅当 AAD-DS 中提供密码哈希时，才可以进行 SSH Kerberos 身份验证，否则该用户无法通过 SSH 连接到群集 |

若要通过 SSH 连接到已加入域的 VM，或者要运行 `kinit` 命令，你需要提供密码。 SSH Kerberos 身份验证要求 AAD-DS 中存在哈希。 如果只想将 SSH 用于管理方案，则可创建一个纯云帐户，并使用该帐户通过 SSH 连接到群集。 其他本地用户仍可使用 Ambari、HDInsight 工具或 HTTP 基本身份验证，而不需要在 AAD-DS 中有密码哈希。

如果你的组织未将密码哈希同步到 AAD-DS，则最佳做法是在 Azure AD 中创建一个仅限云的用户，并在创建群集时将其分配为群集管理员，并将其用于管理目的，包括通过 SSH 获得对 VM 的根访问权限。

若要排查身份验证问题，请参阅此[指南](/hdinsight/domain-joined/domain-joined-authentication-issues)。

## <a name="clients-using-oauth-to-connect-to-hdinsight-gateway-with-hib"></a>客户端使用 OAuth 连接到设置了 HIB 的 HDInsight 网关

在 HIB 设置中，可以更新连接到网关的自定义应用和客户端，以便首先获取所需的 OAuth 令牌。 你可以按照此[文档](/storage/common/storage-auth-aad-app)中的步骤使用以下信息获取令牌：

*   OAuth 资源 URI：`https://hib.azurehdinsight.cn` 
* AppId：7865c1d2-f040-46cc-875f-831a1ef6a28a
*   权限：（名称：Cluster.ReadWrite，id：8f89faa0-ffef-4007-974d-4989b39ad77d）

获取 OAuth 令牌后，可以在向群集网关（例如 https://<clustername>-int.azurehdinsight.cn）发出的 HTTP 请求的授权标头中使用该令牌。 例如，Apache livy API 的示例 curl 命令可能如下所示：
    
```bash
curl -k -v -H "Authorization: Bearer Access_TOKEN" -H "Content-Type: application/json" -X POST -d '{ "file":"wasbs://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn/data/SparkSimpleTest.jar", "className":"com.microsoft.spark.test.SimpleFile" }' "https://<clustername>-int.azurehdinsight.cn/livy/batches" -H "X-Requested-By:<username@domain.com>"
``` 

## <a name="next-steps"></a>后续步骤

* [使用 Azure Active Directory 域服务配置具有企业安全性套餐的 HDInsight 群集](apache-domain-joined-configure-using-azure-adds.md)
* [将 Azure Active Directory 用户同步到 HDInsight 群集](../hdinsight-sync-aad-users-to-cluster.md)
* [监视群集性能](../hdinsight-key-scenarios-to-monitor.md)
