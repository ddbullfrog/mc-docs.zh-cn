---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: REST API 1.2 - Azure Databricks
description: 了解 Databricks REST API 1.2。
ms.openlocfilehash: 0ccc7ea9340d0b2c1345c1e4784b0ec28a5e2395
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937802"
---
# <a name="rest-api-12"></a><a id="rest-api-12"> </a><a id="rest-api-v1"> </a>REST API 1.2

如果不使用 Web UI，可以通过 Azure Databricks REST API 以编程方式访问 Azure Databricks。

本文介绍 REST API 1.2。 对于大多数用例，我们建议使用 [REST API 2.0](../latest/index.md)。 此版本支持 1.2 API 的大多数功能，并支持其他功能。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](../latest/authentication.md)。

## <a name="rest-api-use-cases"></a>REST API 用例

* 启动从现有生产系统或[工作流系统](../../data-pipelines.md)触发的 Apache Spark 作业。
* 以编程方式在一天的固定时间启动某种大小的群集，并在夜间将其关闭。

## <a name="api-categories"></a>API 类别

* 执行上下文：创建可在其中调用 Spark 命令的唯一变量命名空间。
* 命令执行：在特定的执行上下文中运行命令。

## <a name="details"></a>详细信息

* 此 REST API 通过 HTTPS 运行。
* 若要检索信息，请使用 HTTP GET。
* 若要修改状态，请使用 HTTP POST。
* 若要上传文件，请使用 `multipart/form-data`。 否则使用 `application/json`。
* 响应内容类型为 JSON。
* 使用基本身份验证对每个 API 调用用户进行身份验证。
* 用户凭据经过 base64 编码，位于每个 API 调用的 HTTP 标头中。 例如，`Authorization: Basic YWRtaW46YWRtaW4=`。

## <a name="get-started"></a>入门

在以下示例中，请将 `<databricks-instance>` 替换为 Azure Databricks 部署的[工作区 URL](../../../workspace/workspace-details.md#workspace-url)。

### <a name="test-your-connection"></a>测试连接

```bash
> telnet <databricks-instance> 443

Trying 52.11.163.202...
Connected to <databricks-instance>.
Escape character is '^]'.

> nc -v -z <databricks-instance> 443
found 1 connections:
     1: flags=82<CONNECTED,PREFERRED>
    outif utun0
    src x.x.x.x port 59063
    dst y.y.y.y port 443
    rank info not available
    TCP aux info available

Connection to <databricks-instance> port 443 [TCP/HTTPS] succeeded!
```

可以使用上述任一工具来测试连接。 端口 443 是默认的 HTTPS 端口，可在此端口上运行 REST API。 如果无法连接到端口 443，请使用帐户 URL 联系 [help@databricks.com](mailto:help@databricks.com)。

### <a name="sample-api-calls"></a>示例 API 调用

以下示例提供了一些 cURL 命令，但你也可以使用以所选编程语言编写的 HTTP 库。

#### <a name="get-request"></a>GET 请求

> [!NOTE]
>
> 如果 URL 中包含 `&` 字符，则必须将该 URL 括在引号中，以免 UNIX 将其解释为命令分隔符：
>
> ```bash
> curl -n 'https://<databricks-instance>/api/1.2/commands/status?clusterId=batVenom&contextId=35585555555555&commandId=45382422555555555'
> ```

#### <a name="post-request-with-applicationjson"></a>使用 application/json 的 POST 请求

```bash
curl -X POST -n https://<databricks-instance>/api/1.2/contexts/create -d "language=scala&clusterId=batVenom"
```

## <a name="api-endpoints-by-category"></a>按类别列出的 API 终结点

### <a name="execution-context"></a><a id="execution-context"> </a><a id="execution-context-api"> </a>执行上下文

* `https://<databricks-instance>/api/1.2/contexts/create` – 按给定的编程语言在指定的群集上创建执行上下文
  * 使用 application/json 的 POST 请求：
    * data

      ```json
      {"language": "scala", "clusterId": "peaceJam"}
      ```

* `https://<databricks-instance>/api/1.2/contexts/status` – 显示现有执行上下文的状态
  * GET 请求：
    * 示例参数：`clusterId=peaceJam&contextId=179365396413324`
    * `status`: `["Pending", "Running", "Error"]`
* `https://<databricks-instance>/api/1.2/contexts/destroy` – 销毁执行上下文
  * 使用 application/json 的 POST 请求：
    * data

      ```json
      {"contextId" : "1793653964133248955", "clusterId" : "peaceJam"}
      ```

### <a name="command-execution"></a>命令执行

已知限制：命令执行不支持 `%run`。

* `https://<databricks-instance>/api/1.2/commands/execute` – 运行命令或文件。
  * 使用 application/json 的 POST 请求：
    * data

      ```json
      {"language": "scala", "clusterId": "peaceJam", "contextId" : "5456852751451433082", "command": "sc.parallelize(1 to 10).collect"}
      ```

  * 使用 multipart/form-data 的 POST 请求：
    * data

      ```json
      {"language": "python", "clusterId": "peaceJam", "contextId" : "5456852751451433082"}
      ```

    * files

      ```json
      {"command": "./myfile.py"}
      ```

* `https://<databricks-instance>/api/1.2/commands/status` – 显示一个命令的状态或结果
  * GET 请求
    * 示例参数：`clusterId=peaceJam&contextId=5456852751451433082&commandId=5220029674192230006`
    * `status`:`["Queued", "Running", "Cancelling", "Finished", "Cancelled", "Error"]`
* `https://<databricks-instance>/api/1.2/commands/cancel` – 取消一个命令
  * 使用 application/json 的 POST 请求：
    * data

      ```json
      {"clusterId": "peaceJam", "contextId" : "5456852751451433082", "commandId" : "2245426871786618466"}
      ```

## <a name="example-upload-and-run-a-spark-jar"></a>示例：上传和运行 Spark JAR

### <a name="upload-a-jar"></a>上传 JAR

1. 使用 [REST API 2.0](../latest/index.md#rest-api-v2) 上传 JAR 并将其附加到群集。

### <a name="run-a-jar"></a>运行 JAR

1. 创建执行上下文。

   ```bash
   curl -X POST -n  https://<databricks-instance>/api/1.2/contexts/create -d "language=scala&clusterId=batVenom"
   ```

   ```json
   {
     "id": "3558513128163162828"
   }
   ```

2. 执行使用 JAR 的命令。

   ```bash
   curl -X POST -n https://<databricks-instance>/api/1.2/commands/execute \
   -d 'language=scala&clusterId=batVenom&contextId=3558513128163162828&command=println(com.databricks.apps.logs.chapter1.LogAnalyzer.processLogFile(sc,null,"dbfs:/somefile.log"))'
   ```

   ```json
   {
     "id": "4538242203822083978"
   }
   ```

3. 检查命令的状态。 如果运行冗长的 Spark 作业，该命令可能不会立即返回。

   ```bash
   curl -n 'https://<databricks-instance>/api/1.2/commands/status?clusterId=batVenom&contextId=3558513128163162828&commandId=4538242203822083978'
   ```

   ```json
   {
      "id": "4538242203822083978",
      "results": {
        "data": "Content Size Avg: 1234, Min: 1234, Max: 1234",
        "resultType": "text"
      },
      "status": "Finished"
   }
   ```