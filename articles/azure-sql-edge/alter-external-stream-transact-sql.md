---
title: ALTER EXTERNAL STREAM (Transact-SQL) - Azure SQL Edge（预览版）
description: 了解 Azure SQL Edge（预览版）中的 ALTER EXTERNAL STREAM 语句
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: v-tawe
ms.reviewer: sstein
origin.date: 05/19/2020
ms.date: 09/25/2020
ms.openlocfilehash: 7df77926ac72a671e052970ba1bbe1f74f8c4aa4
ms.sourcegitcommit: d89eba76d6f14be0b96c8cdf99decc208003e496
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91248463"
---
# <a name="alter-external-stream-transact-sql"></a>ALTER EXTERNAL STREAM (Transact-SQL)

修改外部流的定义。 不允许修改处于运行状态的流式处理作业使用的外部流 

## <a name="syntax"></a>语法

```sql
  ALTER EXTERNAL STREAM external_stream_name 
  SET 
    [DATA_SOURCE] = <data_source_name> 
    , LOCATION = <location_name> 
    , EXTERNAL_FILE_FORMAT = <external_file_format_name> 
    , INPUT_OPTIONS = <input_options> 
    , OUTPUT_OPTIONS = <output_options> 
```

## <a name="arguments"></a>参数

有关 Alter External Stream 命令参数的详细信息，请参阅 [CREATE EXTERNAL STREAM (Transact-SQL)](create-external-stream-transact-sql.md)。

## <a name="return-code-values"></a>返回代码值

如果成功，ALTER EXTERNAL STREAM 将返回 0。 非零返回值指示失败。


## <a name="see-also"></a>另请参阅

- [CREATE EXTERNAL STREAM (Transact-SQL)](create-external-stream-transact-sql.md) 
- [DROP EXTERNAL STREAM (Transact-SQL)](drop-external-stream-transact-sql.md) 
