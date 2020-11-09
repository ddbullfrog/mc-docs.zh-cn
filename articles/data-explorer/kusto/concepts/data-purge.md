---
title: 数据清除 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的数据清除。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: kedamari
ms.service: data-explorer
ms.topic: reference
origin.date: 05/12/2020
ms.date: 10/29/2020
ms.openlocfilehash: 1de820fe5f9887b34ea57a335b23df3d4a4dbde8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103560"
---
# <a name="data-purge"></a>数据清除

[!INCLUDE [gdpr-intro-sentence](../../includes/gdpr-intro-sentence.md)]

作为数据平台，Azure 数据资源管理器支持通过使用 Kusto `.purge` 和相关命令删除各个记录的功能。 你还可以[清除整个表](#purging-an-entire-table)。  

> [!WARNING]
> 通过 `.purge` 命令删除数据旨在保护个人数据，不应在其他情况下使用。 它不是为支持频繁的删除请求或删除大量数据而设计的，可能会对服务的性能产生显著影响。

## <a name="purge-guidelines"></a>清除准则

在 Azure 数据资源管理器中存储个人数据之前，请仔细设计你的数据架构并调查相关政策。

1. 在最佳方案中，此数据的保留期足够短，系统会自动删除此数据。
1. 如果无法使用保留期，请将受隐私规则限制的所有数据隔离在很少数量的 Azure 数据资源管理器表中。 最好只使用一个表并从所有其他表链接到它。 这种隔离使你可以在包含敏感数据的少量表中运行数据[清除过程](#purge-process)，避开所有其他表。
1. 调用方在批量执行 `.purge` 命令时，应尽量遵循每天每个表 1-2 个命令的要求。 不要发出多个包含不同用户标识谓词的命令， 而应发送单个命令，并在其谓词中包含需要清除的所有用户标识。

## <a name="purge-process"></a>清除过程

有选择性地从 Azure 数据资源管理器中清除数据的过程通过以下步骤进行：

1. 阶段 1：提供包含 Azure 数据资源管理器表名和每记录谓词的输入，指示要删除的记录。 Kusto 会扫描表，尝试标识将参与数据清除过程的数据区。 所标识的区包含符合谓词条件的一个或多个记录。
1. 阶段 2：（软删除）将表中的每个数据区（在步骤 (1) 中标识）替换为重新引入的版本。 重新引入的版本不应有符合谓词条件的记录。 如果没有向表中引入新数据，则在本阶段结束时，查询将不再返回符合谓词条件的数据。 清除软删除阶段的持续时间取决于以下参数： 
     * 必须清除的记录数 
     * 群集的数据区中的记录分布情况 
     * 群集中的节点数  
     * 它为清除操作准备的备用容量
     * 几个其他因素：阶段 2 的持续时间可能短至几秒，也可能长达数小时。
1. 阶段 3：（硬删除）复查可能包含“有害”数据的所有存储项目，并从存储中删除它们。 此阶段至少要在前一阶段完成 5 天后执行，但要在初始命令执行后的 30 天内执行。 设置这些时间线是为了满足数据隐私要求。

发出 `.purge` 命令会触发此过程，此过程需要几天的时间才能完成。 如果符合谓词条件的记录的密度足够大，则此过程将有效地重新引入表中的所有数据。 此重新引入会显著影响性能和 COGS。

## <a name="purge-limitations-and-considerations"></a>清除限制和注意事项

* 清除过程是最终的且不可逆的。 无法撤消此过程或恢复已清除的数据。 [undo table drop](../management/undo-drop-table-command.md) 之类的命令无法恢复已清除的数据。 无法将数据回退到执行最新清除命令之前存在的版本。

* 在运行清除之前，请通过运行查询并检查结果是否与预期结果匹配来验证谓词。 你还可以使用会返回要清除的预期记录数的两步过程。 

* `.purge` 命令针对数据管理终结点 `https://ingest-[YourClusterName].[region].kusto.chinacloudapi.cn` 来执行。
   该命令要求对相关数据库具有[数据库管理](../management/access-control/role-based-authorization.md)权限。 
* 由于清除过程的性能影响，并且要保证遵守[清除准则](#purge-guidelines)，调用方需要修改数据架构，尽量减少包含相关数据的表的数量，并针对每个表批量处理命令，以减轻清除过程中产生的较大的 COGS 影响。
* [.purge](#purge-table-tablename-records-command) 命令的 `predicate` 参数用于指定要清除的记录。
`Predicate` 大小限制为 63 KB。 在构造 `predicate` 时需遵循以下要求：
    * 使用[“in”运算符](../query/inoperator.md)，例如 `where [ColumnName] in ('Id1', 'Id2', .. , 'Id1000')`。 
    * 请注意[“in”运算符](../query/inoperator.md)的限制（列表最多可以包含 `1,000,000` 个值）。
    * 如果查询太大，请使用 [`externaldata` 运算符](../query/externaldata-operator.md)，例如 `where UserId in (externaldata(UserId:string) ["https://...blob.core.chinacloudapi.cn/path/to/file?..."])`。 此文件存储着要清除的 ID 的列表。
    * 在展开所有 `externaldata` blob 后，总查询大小（所有 blob 的总大小）不能超过 64 MB。 

## <a name="purge-performance"></a>清除性能

在任何给定时间，都只能在群集上执行一个清除请求。 所有其他请求将以 `Scheduled` 状态排入队列。 监视清除请求队列大小，并使其保持在适当的限制范围内，以满足数据的相应要求。

若要减少清除执行时间，请执行以下操作：
* 按照[清除准则](#purge-guidelines)减少清除数据量。
* 调整[缓存策略](../management/cachepolicy.md)，因为清除冷数据花费的时间更长。
* 横向扩展群集

* 仔细考虑后，增加群集清除容量，详见[盘区清除重新生成容量](../management/capacitypolicy.md#extents-purge-rebuild-capacity)。 更改此参数需要创建一个[支持票证](https://portal.azure.cn/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)

## <a name="trigger-the-purge-process"></a>触发清除过程

> [!NOTE]
> 清除执行是通过对数据管理终结点 https://ingest- [YourClusterName].[Region].kusto.chinacloudapi.cn 运行 [purge table *TableName* records](#purge-table-tablename-records-command) 命令来调用的。

### <a name="purge-table-tablename-records-command"></a>Purge table TableName records 命令

可以通过两种方式调用 Purge 命令，具体取决于使用方案：

* 编程调用：要由应用程序调用的单个步骤。 调用此命令将直接触发清除执行序列。

  **语法**

  ```kusto
  // Connect to the Data Management service
  #connect "https://ingest-[YourClusterName].[region].kusto.chinacloudapi.cn" 
 
  .purge table [TableName] records in database [DatabaseName] with (noregrets='true') <| [Predicate]
   ```

* 人为调用：一个两步过程，需要显式确认为单个步骤。 第一次调用此命令将返回验证令牌，需要提供该令牌才能运行实际清除操作。 此序列降低了无意中删除错误数据的风险。

 > [!NOTE]
 > 两步调用中的第一步需要对整个数据集运行查询，以标识要清除的记录。
 > 对于大型表（尤其是有大量冷缓存数据的表），此查询可能会超时或失败。 如果失败，请自行验证谓词，并在验证正确性后使用带 `noregrets` 选项的单步清除。

**语法**

  ```kusto
     // Connect to the Data Management service
     #connect "https://ingest-[YourClusterName].[region].kusto.chinacloudapi.cn" 
     
     // Step #1 - retrieve a verification token (no records will be purged until step #2 is executed)
     .purge table [TableName] records in database [DatabaseName] <| [Predicate]

     // Step #2 - input the verification token to execute purge
     .purge table [TableName] records in database [DatabaseName] with (verificationtoken='<verification token from step #1>') <| [Predicate]
  ```
    
| parameters  | 说明  |
|---------|---------|
| `DatabaseName`   |   数据库的名称      |
| `TableName`     |     表名称    |
| `Predicate`    |    标识要清除的记录。 请参阅下文中的“Purge 谓词限制”。 | 
| `noregrets`    |     如果设置了此项，则会触发单步激活。    |
| `verificationtoken`     |  在两步激活方案（不设置 `noregrets`）中，此令牌可用于执行第二步并提交操作。 如果未指定 `verificationtoken`，它将触发此命令的第一步。 将返回有关清除的信息，其中包含应当传回给命令以执行步骤 2 的一个令牌。   |

**Purge 谓词限制**

* 谓词必须是一个简单的选择（例如， *where [ColumnName] == 'X'*  / *where [ColumnName] in ('X', 'Y', 'Z') and [OtherColumn] == 'A'* ）。
* 多个筛选器必须使用“and”进行组合，而不是使用单独的 `where` 子句（例如，请使用 `where [ColumnName] == 'X' and  OtherColumn] == 'Y'` 而非 `where [ColumnName] == 'X' | where [OtherColumn] == 'Y'`）。
* 谓词不能引用正在清除的表 ( *TableName* ) 以外的表。 谓词只能包含选择语句 (`where`)。 它无法投影表中的特定列（运行“ *`table` | Predicate* ”时的输出架构必须与表架构匹配）。
* 不支持系统函数（例如 `ingestion_time()`、`extent_id()`）。

#### <a name="example-two-step-purge"></a>示例：两步清除

若要在两步激活方案中开始清除，请运行以下命令的步骤 1：

 ```kusto
    // Connect to the Data Management service
     #connect "https://ingest-[YourClusterName].[region].kusto.chinacloudapi.cn" 
     
    .purge table MyTable records in database MyDatabase <| where CustomerId in ('X', 'Y')
 ```

**输出**

 | NumRecordsToPurge | EstimatedPurgeExecutionTime| VerificationToken
 |---|---|---
 | 1,596 | 00:00:02 | e43c7184ed22f4f23c7a9d7b124d196be2e570096987e5baadf65057fa65736b

然后，在运行步骤 2 之前验证 NumRecordsToPurge。 

若要在两步激活方案中完成清除，请使用从步骤 1 返回的验证令牌来运行步骤 2：

```kusto
.purge table MyTable records in database MyDatabase
 with(verificationtoken='e43c7184ed22f4f23c7a9d7b124d196be2e570096987e5baadf65057fa65736b')
<| where CustomerId in ('X', 'Y')
```

**输出**

| `OperationId` | `DatabaseName` | `TableName`|`ScheduledTime` | `Duration` | `LastUpdatedOn` |`EngineOperationId` | `State` | `StateDetails` |`EngineStartTime` | `EngineDuration` | `Retries` |`ClientRequestId` | `Principal`|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| c9651d74-3b80-4183-90bb-bbe9e42eadc4 |MyDatabase |MyTable |2019-01-20 11:41:05.4391686 |00:00:00.1406211 |2019-01-20 11:41:05.4391686 | |计划 | | | |0 |KE.RunCommand;1d0ad28b-f791-4f5a-a60f-0e32318367b7 |AAD app id=...|

#### <a name="example-single-step-purge"></a>示例：单步清除

若要在单步激活方案中触发清除，请运行以下命令：

```kusto
// Connect to the Data Management service
  #connect "https://ingest-[YourClusterName].[region].kusto.chinacloudapi.cn" 
 
.purge table MyTable records in database MyDatabase with (noregrets='true') <| where CustomerId in ('X', 'Y')
```

**输出**

| `OperationId` |`DatabaseName` |`TableName` |`ScheduledTime` |`Duration` |`LastUpdatedOn` |`EngineOperationId` |`State` |`StateDetails` |`EngineStartTime` |`EngineDuration` |`Retries` |`ClientRequestId` |`Principal`|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| c9651d74-3b80-4183-90bb-bbe9e42eadc4 |MyDatabase |MyTable |2019-01-20 11:41:05.4391686 |00:00:00.1406211 |2019-01-20 11:41:05.4391686 | |计划 | | | |0 |KE.RunCommand;1d0ad28b-f791-4f5a-a60f-0e32318367b7 |AAD app id=...|

### <a name="cancel-purge-operation-command"></a>Cancel purge operation 命令

如果需要，你可以取消挂起的清除请求。

> [!NOTE]
> 此操作用于错误恢复方案。 这并不保证成功，不应当用作正常操作流程的一部分。 它只能应用于队列内请求（尚未调度到引擎节点来执行的请求）。 此命令在数据管理终结点上执行。

**语法**

```kusto
 .cancel purge <OperationId>
```

**示例**

```kusto
 .cancel purge aa894210-1c60-4657-9d21-adb2887993e1
```

**输出**

此命令的输出与“show purges *OperationId* ”命令的输出相同，它显示正在取消的清除操作的更新后状态。 如果尝试成功，则操作状态将更新为 `Abandoned`。 否则，操作状态不会更改。 

|`OperationId` |`DatabaseName` |`TableName` |`ScheduledTime` |`Duration` |`LastUpdatedOn` |`EngineOperationId` |`State` |`StateDetails` |`EngineStartTime` |`EngineDuration` |`Retries` |`ClientRequestId` |`Principal`
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|c9651d74-3b80-4183-90bb-bbe9e42eadc4 |MyDatabase |MyTable |2019-01-20 11:41:05.4391686 |00:00:00.1406211 |2019-01-20 11:41:05.4391686 | |已放弃 | | | |0 |KE.RunCommand;1d0ad28b-f791-4f5a-a60f-0e32318367b7 |AAD app id=...

## <a name="track-purge-operation-status"></a>跟踪清除操作状态 

> [!NOTE]
> 可以通过 [show purges](#show-purges-command) 命令来跟踪清除操作，该命令针对数据管理终结点 https://ingest- [YourClusterName].[region].kusto.chinacloudapi.cn 来执行。

Status = 'Completed' 表示清除操作的第一阶段成功完成，也就是说，记录已软删除，再也不能对其进行查询。 客户不需要跟踪和验证第二阶段（硬删除）的完成情况。 此阶段由 Azure 数据资源管理器在内部进行监视。

### <a name="show-purges-command"></a>Show purges 命令

`Show purges` 命令通过指定操作 ID 来显示所请求的时间段内的清除操作状态。 

```kusto
.show purges <OperationId>
.show purges [in database <DatabaseName>]
.show purges from '<StartDate>' [in database <DatabaseName>]
.show purges from '<StartDate>' to '<EndDate>' [in database <DatabaseName>]
```

| 属性  |说明  |必需/可选|
|---------|------------|-------|
|`OperationId `   |      在执行单个阶段或第二阶段后输出的数据管理操作 ID。   |必需
|`StartDate`    |   用于筛选操作的时间限制起点。 如果省略，则默认为当前时间之前的 24 小时。      |可选
|`EndDate`    |  用于筛选操作的时间限制终点。 如果省略，则默认为当前时间。       |可选
|`DatabaseName`    |     用于筛选结果的数据库名称。    |可选

> [!NOTE]
> 只会为客户端对其具有[数据库管理员](../management/access-control/role-based-authorization.md)权限的数据库提供状态。

**示例**

```kusto
.show purges
.show purges c9651d74-3b80-4183-90bb-bbe9e42eadc4
.show purges from '2018-01-30 12:00'
.show purges from '2018-01-30 12:00' to '2018-02-25 12:00'
.show purges from '2018-01-30 12:00' to '2018-02-25 12:00' in database MyDatabase
```

**输出** 

|`OperationId` |`DatabaseName` |`TableName` |`ScheduledTime` |`Duration` |`LastUpdatedOn` |`EngineOperationId` |`State` |`StateDetails` |`EngineStartTime` |`EngineDuration` |`Retries` |`ClientRequestId` |`Principal`
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|c9651d74-3b80-4183-90bb-bbe9e42eadc4 |MyDatabase |MyTable |2019-01-20 11:41:05.4391686 |00:00:33.6782130 |2019-01-20 11:42:34.6169153 |a0825d4d-6b0f-47f3-a499-54ac5681ab78 |已完成 |清除已成功完成（存储项目待删除） |2019-01-20 11:41:34.6486506 |00:00:04.4687310 |0 |KE.RunCommand;1d0ad28b-f791-4f5a-a60f-0e32318367b7 |AAD app id=...

* `OperationId` - 执行清除时返回的 DM 操作 ID。 
* `DatabaseName`** - 数据库名称（区分大小写）。 
* `TableName` - 表名称（区分大小写）。 
* `ScheduledTime` - 对 DM 服务执行清除命令的时间。 
* `Duration` - 清除操作的总持续时间，包括执行 DM 队列等待时间。 
* `EngineOperationId` - 正在引擎中执行的实际清除的操作 ID。 
* `State` - 清除状态，可以是以下值之一： 
    * `Scheduled` - 已计划执行清除操作。 如果作业一直处于 Scheduled 状态，则可能存在积压的清除操作。 若要清除此积压的操作，请参阅[清除性能](#purge-performance)。 如果清除操作因暂时性错误而失败，则 DM 会重试该操作，并将其重新设置为 Scheduled（因此，你可能会看到某个操作从 Scheduled 转变为 InProgress，然后又转变回 Scheduled）。
    * `InProgress` - 清除操作正在引擎中进行。 
    * `Completed` - 清除已成功完成。
    * `BadInput` - 清除因错误的输入而失败，并且不会重试。 此失败可能是由各种问题引起的，例如，谓词中出现语法错误、清除命令的谓词无效、查询超出限制（例如，`externaldata` 运算符中的实体超出 1 百万个，或总展开查询大小超出 64 MB）、`externaldata` blob 出现 404 或 403 错误。
    * `Failed` - 清除失败，并且不会重试。 如果操作在队列中等待的时间太长（超过 14 天），则可能会发生这种失败，原因是积压了其他的清除操作，或者失败数超出了重试限制。 后者将引发内部监视警报，并将由 Azure 数据资源管理器团队进行调查。 
* `StateDetails` - 状态说明。
* `EngineStartTime` - 向引擎发出命令的时间。 如果此时间与 ScheduledTime 之间存在很大的差异，通常表明积压了大量的清除操作，并且群集没有跟上节奏。 
* `EngineDuration` - 在引擎中实际执行清除操作的时间。 如果多次重试了清除，则是所有执行持续时间的总和。 
* `Retries` - 由于暂时性错误，DM 服务重试操作的次数。
* `ClientRequestId` - DM 清除请求的客户端活动 ID。 
* `Principal` - 清除命令发出者的标识。

## <a name="purging-an-entire-table"></a>清除整个表

清除表包括删除表，并将其标记为已清除，以便[清除过程](#purge-process)中描述的硬删除过程在其上运行。 如果删除表但不清除它，则不会删除其所有存储项目。 将根据最初在表上设置的硬保留策略来删除这些项目。 如果适合你的方案，`purge table allrecords` 命令将很快速高效，并且更适合在清除记录过程中使用。 

> [!NOTE]
> 此命令是通过对数据管理终结点 https://ingest- [YourClusterName].[region].kusto.chinacloudapi.cn 运行 [purge table *TableName* allrecords](#purge-table-tablename-allrecords-command) 命令来调用的。

### <a name="purge-table-tablename-allrecords-command"></a>Purge table *TableName* allrecords 命令

此命令类似于“[.purge table records ](#purge-table-tablename-records-command)”命令，可以通过编程模式（单步）或手动（两步）模式调用。

1. 编程调用（单步）：

     **语法**

     ```kusto
     // Connect to the Data Management service
     #connect "https://ingest-[YourClusterName].[Region].kusto.chinacloudapi.cn" 
     
     .purge table [TableName] in database [DatabaseName] allrecords with (noregrets='true')
     ```

1. 人工调用（两步）：

     **语法**

     ```kusto
   
     // Connect to the Data Management service
     #connect "https://ingest-[YourClusterName].[Region].kusto.chinacloudapi.cn" 
     
     // Step #1 - retrieve a verification token (the table will not be purged until step #2 is executed)

     .purge table [TableName] in database [DatabaseName] allrecords

     // Step #2 - input the verification token to execute purge
     .purge table [TableName] in database [DatabaseName] allrecords with (verificationtoken='<verification token from step #1>')
     ```

    | parameters  |说明  |
    |---------|---------|
    | `DatabaseName`   |   数据库的名称。      |
    | `TableName`    |     表的名称。    |
    | `noregrets`    |     如果设置了此项，则会触发单步激活。    |
    | `verificationtoken`     |  在两步激活方案（不设置 `noregrets`）中，此令牌可用于执行第二步并提交操作。 如果未指定 `verificationtoken`，它将触发此命令的第一步。 在此步骤中，将返回一个可传回给命令并执行步骤 2 的令牌。|

#### <a name="example-two-step-purge"></a>示例：两步清除

1. 若要在两步激活方案中开始清除，请运行以下命令的步骤 1： 

    ```kusto
    // Connect to the Data Management service
     #connect "https://ingest-[YourClusterName].[Region].kusto.chinacloudapi.cn" 
     
    .purge table MyTable in database MyDatabase allrecords
    ```

    **输出**

    | `VerificationToken`|
    |---|
    | e43c7184ed22f4f23c7a9d7b124d196be2e570096987e5baadf65057fa65736b|

1.  若要在两步激活方案中完成清除，请使用从步骤 1 返回的验证令牌来运行步骤 2：

    ```kusto
    .purge table MyTable in database MyDatabase allrecords 
    with (verificationtoken='eyJTZXJ2aWNlTmFtZSI6IkVuZ2luZS1pdHNhZ3VpIiwiRGF0YWJhc2VOYW1lIjoiQXp1cmVTdG9yYWdlTG9ncyIsIlRhYmxlTmFtZSI6IkF6dXJlU3RvcmFnZUxvZ3MiLCJQcmVkaWNhdGUiOiIgd2hlcmUgU2VydmVyTGF0ZW5jeSA9PSAyNSJ9')
    ```
    
    输出与“.show tables”命令输出相同（不返回已清除的表）。

    **输出**

    |  TableName|DatabaseName|文件夹|DocString
    |---|---|---|---
    |  OtherTable|MyDatabase|---|---


#### <a name="example-single-step-purge"></a>示例：单步清除

若要在单步激活方案中触发清除，请运行以下命令：

```kusto
// Connect to the Data Management service
#connect "https://ingest-[YourClusterName].[Region].kusto.chinacloudapi.cn" 

.purge table MyTable in database MyDatabase allrecords with (noregrets='true')
```

输出与“.show tables”命令输出相同（不返回已清除的表）。

**输出**

|TableName|DatabaseName|文件夹|DocString
|---|---|---|---
|OtherTable|MyDatabase|---|---

