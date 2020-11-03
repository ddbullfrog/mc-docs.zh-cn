---
title: .execute database script - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 .execute database script 功能。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 06/14/2020
ms.date: 10/29/2020
ms.openlocfilehash: 9ed87c68ab843cc8b1f8f299eff0804e2c9c0453
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106011"
---
# <a name="execute-database-script"></a>.execute database script

在单个数据库作用域内批量执行控制命令。

## <a name="syntax"></a>语法

`.execute` `database` `script`  
[`with` `(` PropertyName `=` PropertyValue [`,` ...] `)`]    
`<|`  
 *Control-commands-script*

### <a name="parameters"></a>parameters

* *Control-commands-script* ：包含一个或多个控制命令的文本。
* *数据库作用域* ：脚本应用于作为请求上下文一部分指定的“数据库作用域”。

### <a name="optional-properties"></a>可选属性

| 属性            | 类型            | 说明                          |
|---------------------|-----------------|---------------------------------------------------------------------------------------------------|
| `ContinueOnErrors`            | `bool`        | 如果设置为 `false`，脚本将在发生第一个错误时停止。 如果设置为 `true`，脚本将继续执行。 默认：`false`。 |

## <a name="output"></a>输出

脚本中显示的每个命令在输出表中都显示为一条单独记录。 每个记录都具有以下字段：

|输出参数 |类型 |说明
|---|---|--- 
|OperationId  |Guid |命令的标识符。
|CommandType  |String |命令的类型。
|CommandText  |String |特定命令的文本。
|结果|String|特定命令执行的结果。
|Reason|String|有关命令执行结果的详细信息。

>[!NOTE]
>* 脚本文本可能包括命令之间的空行和注释。
>* 命令按它们在输入脚本中出现的顺序依次执行。
>* 脚本执行是非事务性的，出错时不执行回滚。 建议在使用 `.execute database script` 时使用幂等形式的命令。
>* 命令的默认行为 - 在第一个错误处失败，可以使用属性参数对其进行更改。
>* 不执行只读控制命令（.show 命令），并报告状态 `Skipped`。

## <a name="example"></a>示例

```kusto
.execute database script <|
//
// Create tables
.create-merge table T(a:string, b:string)
//
// Apply policies
.alter-merge table T policy retention softdelete = 10d 
//
// Create functions
.create-or-alter function
  with (skipvalidation = "true") 
  SampleT1(myLimit: long) { 
    T1 | limit myLimit
}
```

|OperationId|CommandType|CommandText|结果|Reason|
|---|---|---|---|---|
|1d28531b-58c8-4023-a5d3-16fa73c06cfa|TableCreate|.create-merge table T(a:string, b:string)|已完成||
|67d0ea69-baa4-419a-93d3-234c03834360|RetentionPolicyAlter|.alter-merge table T policy retention softdelete = 10d|已完成||
|0b0e8769-d4e8-4ff9-adae-071e52a650c7|FunctionCreateOrAlter|.create-or-alter function<br>with (skipvalidation = "true")<br>SampleT1(myLimit: long) {<br>T1 \| limit myLimit<br>}|已完成||
