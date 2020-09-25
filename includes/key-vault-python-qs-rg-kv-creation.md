---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
origin.date: 09/03/2020
ms.date: 09/16/2020
ms.author: v-tawe
ms.openlocfilehash: 22a1cc91a858e7ed29b850052f88ee5e97ab8c6c
ms.sourcegitcommit: 39410f3ed7bdeafa1099ba5e9ec314b4255766df
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2020
ms.locfileid: "90678363"
---
1. 使用 `az group create` 命令以创建资源组：

    ```azurecli
    az group create --name KeyVault-PythonQS-rg --location chinaeast
    ```

    如果愿意，你可以将“chinaeast”更改为离你更近的位置。

1. 使用 `az keyvault create` 创建密钥保管库：

    ```azurecli
    az keyvault create --name <your-unique-keyvault-name> --resource-group KeyVault-PythonQS-rg
    ```

    将 `<your-unique-keyvault-name>` 替换为在整个 Azure 中均唯一的名称。 通常使用个人或公司名称以及其他数字和标识符。 

1. 创建用于向代码提供 Key Vault 名称的环境变量：

    # <a name="cmd"></a>[cmd](#tab/cmd)

    ```cmd
    set KEY_VAULT_NAME=<your-unique-keyvault-name>
    ```

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash
    export KEY_VAULT_NAME=<your-unique-keyvault-name>
    ```

    ---
