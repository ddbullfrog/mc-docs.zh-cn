---
title: include 文件
description: include 文件
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 03/15/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 15720029bf716c0b732ff4d2f3f8e04b069f9d78
ms.sourcegitcommit: 39410f3ed7bdeafa1099ba5e9ec314b4255766df
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2020
ms.locfileid: "90680071"
---
<!-- This is the note explaining about the avro and json formats when routing to blob storage. -->
> [!NOTE]
> 可以将数据以 [Apache Avro](https://avro.apache.org/) 格式（默认）或 JSON 格式（预览版）写入 Blob 存储。 
>    
> 对 JSON 格式进行编码的功能在除“中国东部”和“中国北部”以外的可以使用 IoT 中心的所有区域均为预览版。 编码格式只能在配置 Blob 存储终结点时设置。 不能更改已设置的终结点的格式。 使用 JSON 编码时，必须在消息系统属性中将 contentType 设置为 JSON，将 contentEncoding 设置为 UTF-8。 
>
> 若要更详细地了解如何使用 Blob 存储终结点，请参阅[有关如何路由到存储的指南](../articles/iot-hub/iot-hub-devguide-messages-d2c.md#azure-storage)。
>