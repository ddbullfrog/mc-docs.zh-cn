---
title: 列管理 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的列管理。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/24/2020
ms.openlocfilehash: b66836aa2f8c2f8dd3763dab2daaa630fbc8f231
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146436"
---
# <a name="columns-management"></a>列管理

本部分描述了以下用于管理表列的控制命令：

|命令 |说明 |
|------- | -------|
|[alter column](alter-column.md) |更改现有表列的数据类型 |
|[alter-merge table column](alter-merge-table-column.md) 和 [alter table column-docstrings](alter-merge-table-column.md#alter-table-column-docstrings) | 设置指定表的一列或多列的 `docstring` 属性
|[`.alter table`](alter-table-command.md), [`.alter-merge table`](alter-table-command.md) | 修改表的架构（添加/删除列） |
|[drop column 和 drop table columns](drop-column.md) |从表中删除一列或多列 |
|[rename column or columns](rename-column.md) |更改现有或多个表列的名称 |
