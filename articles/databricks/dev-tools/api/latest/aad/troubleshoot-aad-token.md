---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/07/2020
title: Azure Active Directory 访问令牌疑难解答 - Azure Databricks
description: 了解如何对 Azure Active Directory (Azure AD) 令牌进行故障排除和验证，以便能够访问 Databricks REST API。
ms.openlocfilehash: 5e60f87bb847c6cb0c5eebd70a53b3d2150dc2cf
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937793"
---
# <a name="troubleshoot-azure-active-directory-access-tokens"></a>Azure Active Directory 访问令牌疑难解答

本文介绍如何排查在获取 Azure Active Directory 访问令牌时可能遇到的错误，以及如何验证访问令牌。

## <a name="failed-to-get-token-using-username-and-password"></a>无法使用用户名和密码获取令牌

### <a name="error-message"></a>错误消息

```console
The user or administrator has not consented to use the application with ID <client-id>.
Send an interactive authorization request for this user and resource.
```

### <a name="solution"></a>解决方案

1. 如果 AzureDatabricks 资源未添加到你的应用程序，请要求管理员用户添加它。
2. 使用[交互式方法](app-aad-token.md#interactive)获取令牌。 该网页将指导你向应用程序授予权限。 或者，单击应用程序配置中描述的“授予权限”按钮。 授予权限后，可以使用编程方法获取令牌。

## <a name="redirect-uris-do-not-match"></a>重定向 URI 不匹配

### <a name="error-message"></a>错误消息

```console
The reply url specified in the request does not match the reply urls configured for the application: '<application-id>'
```

### <a name="solution"></a>解决方案

检查请求中的重定向 URI 是否与应用程序中的重定向 URI 相匹配。

## <a name="validate-an-access-token"></a>验证访问令牌

如果有 Azure AD 访问令牌，可以验证它是否包含正确的信息（请参阅[验证令牌](/active-directory/develop/access-tokens#validating-tokens)）。

应该验证以下字段是否与记录匹配：

* **aud**：Azure Databricks 资源 ID：`2ff814a6-3304-4ab8-85cb-cd0e6f879c1d`
* **iss**：应为 `https://sts.chinacloudapi.cn/<tenant-id>/`
* **tid**：应为工作区的租户（通过组织 ID 或工作区设备 ID 查找）
* **nbf/exp**：当前时间应介于 `nbf` 和 `exp` 之间
* **unique_name**：应是 Databricks 工作区中的用户，除非该用户是工作区设备资源的贡献者

使用 [OIDC 终结点](https://login.microsoftonline.com/common/.well-known/openid-configuration)中的公共证书验证令牌签名。

以下代码片段显示了令牌的有效负载。 首先必须使用 `pip install pyjwt` 安装 [PyJWT](https://pyjwt.readthedocs.io/en/latest/installation.html) 库：

```python
import jwt
def decode_token(token):
  decoded = jwt.decode(token, verify=False)

  for key in decoded.keys():
     print key + ': ' + str(decoded[key])
```

如果要对令牌进行完全解码（包括签名验证），可以使用以下代码片段：

```python
Import jwt
import requests
from cryptography.x509 import load_pem_x509_certificate
from cryptography.hazmat.backends import default_backend

PEMSTART = '-----BEGIN CERTIFICATE-----\n'
PEMEND = '\n-----END CERTIFICATE-----\n'

    # get Microsoft Azure public key
def get_public_key_for_token(kid):
  response = requests.get(
  'https://login.microsoftonline.com/common/.well-known/openid-configuration',
  ).json()

  jwt_uri = response['jwks_uri']
  response_keys = requests.get(jwt_uri).json()
  pubkeys = response_keys['keys']

  public_key = ''

  for key in pubkeys:
      # found the key that matching the kid in the token header
      if key['kid'] == kid:
          # construct the public key object
          mspubkey = str(key['x5c'][0])
          cert_str = PEMSTART + mspubkey + PEMEND
          cert_obj = load_pem_x509_certificate(cert_str, default_backend())
          public_key = cert_obj.public_key()

  return public_key

# decode the given Azure AD access token
def aad_access_token_decoder(access_token):
  header = jwt.get_unverified_header(access_token)
  public_key = get_jwt_publickey(header['kid'])
  # the value of the databricks_resource_id is as defined above
  # i.e., databricks_resource_id =  "2ff814a6-3304-4ab8-85cb-cd0e6f879c1d"
  decoded = jwt.decode(access_token, key=public_key, algorithms='RS256',
      audience=<databricks-resource-id>)

  for key in decoded.keys():
      print key + ': ' + str(decoded[key])
```

下面是上述代码片段的输出示例：

> [!div class="mx-imgBorder"]
> ![Azure 已注册的应用的设置](../../../../_static/images/aad/output.png)

还可以通过在线 JWT 解码器查看已解码的令牌（如果它们不敏感）。 在线解码器的示例是 [jwt.ms](https://jwt.ms) 和 [jwt.io](https://jwt.io)。