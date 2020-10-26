---
author: rockboyfor
ms.topic: include
origin.date: 10/09/2019
ms.date: 08/10/2020
ms.author: v-yeche
ms.openlocfilehash: 1c20a3c08115e92c58c18c8f424a42d8b95b4b9e
ms.sourcegitcommit: fce0810af6200f13421ea89d7e2239f8d41890c0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2020
ms.locfileid: "92470211"
---
在 Linux 或[适用于 Linux 的 Windows 子系统][install-wsl]或 MacOS 上的基于 bash 的 shell 中，使用 `curl` 下载 Consul Helm 图表版本，如下所示：

```bash
# Specify the Consul Helm chart version that will be leveraged throughout these instructions
CONSUL_HELM_VERSION=0.10.0

curl -sL "https://github.com/hashicorp/consul-helm/archive/v$CONSUL_HELM_VERSION.tar.gz" | tar xz
mv consul-helm-$CONSUL_HELM_VERSION consul-helm
```

<!-- LINKS - external -->

[install-wsl]: https://docs.microsoft.com/windows/wsl/install-win10

<!-- Update_Description: update meta properties, wording update, update link -->