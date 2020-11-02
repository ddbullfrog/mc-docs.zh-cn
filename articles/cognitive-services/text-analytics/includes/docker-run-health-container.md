---
title: docker run 命令的运行容器示例
titleSuffix: Azure Cognitive Services
description: 运行状况容器的 Docker run 命令
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 08/03/2020
ms.author: v-johya
ms.openlocfilehash: 033a1583ff7b8347dfeaf3c49d7a71ef17b4a61d
ms.sourcegitcommit: caa18677adb51b5321ad32ae62afcf92ac00b40b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/08/2020
ms.locfileid: "92211342"
---
若要运行容器，请先查找其映像 ID：
 
```bash
docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```

执行以下 `docker run` 命令。 将下面的占位符替换为你自己的值：

| 占位符 | Value | 格式或示例 |
|-------------|-------|---|
| **{API_KEY}** | 文本分析资源的密钥。 可以在 Azure 门户中资源的“密钥和终结点”页上找到此项。 |`xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`|
| **{ENDPOINT_URI}** | 用于访问文本分析 API 的终结点。 可以在 Azure 门户中资源的“密钥和终结点”页上找到此项。 | `https://<your-custom-subdomain>.cognitiveservices.azure.com` |
| **{IMAGE_ID}** | 容器的映像 ID。 | `1.1.011300001-amd64-preview` |
| **{INPUT_DIR}** | 容器的输入目录。 | Windows： `C:\healthcareMount` <br> Linux/MacOS：`/home/username/input` |

```bash
docker run --rm -it -p 5000:5000 --cpus 6 --memory 12g \
--mount type=bind,src={INPUT_DIR},target=/output {IMAGE_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Disk:Format=json
```

此命令：

- 假定主机上存在输入目录
- 从容器映像运行运行状况容器的文本分析
- 分配 6 个 CPU 核心和 12 千兆字节 (GB) 内存
- 公开 TCP 端口 5000，并为容器分配伪 TTY
- 退出后自动删除容器。 容器映像在主计算机上仍然可用。

