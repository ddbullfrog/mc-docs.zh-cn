---
title: 时序分析 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的时序分析。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 05/06/2019
ms.date: 10/29/2020
ms.openlocfilehash: 0d2cea766674d04d5451c98c06a32ce1f56f0fb6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104266"
---
# <a name="time-series-analysis"></a>时序分析 

Kusto 查询语言以本机数据类型形式提供系列支持。
make-series 运算符将数据转换为序列数据类型，并提供一系列函数，用于对该数据类型进行高级处理。 这些函数涵盖数据清除、机器学习建模和离群值检测。