---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: 使用 Azure Active Directory 身份验证库获取 Azure Active Directory 令牌 - Azure Databricks
description: 了解如何使用 Azure Active Directory 身份验证库 (ADAL) 获取 Azure Active Directory (Azure AD) 令牌来对 Databricks REST API 进行身份验证。
ms.openlocfilehash: f1393cbed3d59c7d512b4a1eca804947c57d6410
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937803"
---
# <a name="get-an-azure-active-directory-token-using-azure-active-directory-authentication-library"></a>使用 Azure Active Directory 身份验证库获取 Azure Active Directory 令牌

可使用 Azure Active Directory 身份验证库 (ADAL) 以编程方式获取 Azure Active Directory (Azure AD) 访问令牌。 本文通过 Python 示例介绍了 ADAL 库的基本用法以及所需的用户输入。

你还可在 Azure Active Directory 中定义服务主体，并为该服务主体（而不是用户）获取 Azure AD 访问令牌。 请参阅[使用服务主体获取 Azure Active Directory 令牌](service-prin-aad-token.md)。

## <a name="configure-an-app-in-azure-portal"></a>在 Azure 门户中配置应用

1. 在 Azure 门户中向 Azure AD 终结点注册应用程序。 遵循[快速入门：向 Azure Active Directory v1.0 终结点注册应用](/active-directory/develop/quickstart-v1-add-azure-ad-app)。 或者，你可使用已注册的应用。

   在“重定向 URI”字段中，选择“公共客户端/本机(移动和桌面)”并输入重定向 URI。 在下面的示例中，重定向 URI 值为 `http://localhost`。

   > [!div class="mx-imgBorder"]
   > ![注册应用](../../../../_static/images/aad/register-app.png)

2. 单击“注册”。
3. 转到“应用注册”>“查看所有应用程序”，然后选择该应用。 复制“应用程序(客户端) ID”。

   > [!div class="mx-imgBorder"]
   > ![关于 Azure 已注册的应用的概述](../../../../_static/images/aad/registeredapp.png)

4. 向 AzureDatabricks 添加已注册应用程序的必需权限 。 只有管理员用户可执行此步骤。 如果在执行此步骤时遇到“权限”问题，请与管理员联系以获得帮助。
   1. 在应用程序页面上，单击“查看 API 权限”。

      > [!div class="mx-imgBorder"]
      > ![Azure 已注册的应用的设置](../../../../_static/images/aad/registered-app-settings.png)

   1. 单击“添加权限”。

      > [!div class="mx-imgBorder"]
      > ![向应用添加所需权限](../../../../_static/images/aad/app-permissions.png)

   1. 选择“我的组织使用的 API”选项卡，搜索 AzureDatabricks 并将其选中 。

      > [!div class="mx-imgBorder"]
      > ![添加 AzureDatabricks API 权限](../../../../_static/images/aad/azure-databricks-api.png)

   1. 选择“user_impersonation”，然后单击“添加权限” 。

      > [!div class="mx-imgBorder"]
      > ![Azure 应用委派的权限](../../../../_static/images/aad/delegated-permissions.png)

   1. 单击“向 ### 授予管理员同意”，然后单击“是” 。  若要执行此步骤，你必须是管理员用户或有权向应用程序授予同意。 如果跳过此步骤，则必须在首次使用应用程序时，使用[授权代码流（交互式）](#interactive)提供同意。 之后便可使用[用户名-密码流（编程式）](#programmatic)方法。

      > [!div class="mx-imgBorder"]
      > ![向应用权限添加其他用户和组](../../../../_static/images/aad/give-permissions.png)

可将其他用户添加到应用程序。 有关详细信息，请参阅[向 Azure Active Directory 中的应用程序分配用户和组](/active-directory/manage-apps/methods-for-assigning-users-and-groups)。 用户如果没有所需的权限，则无法获取令牌。

## <a name="get-an-azure-active-directory-access-token"></a>获取 Azure Active Directory 访问令牌

若要获取访问令牌，可使用以下任一方法：

* [授权代码流（交互式）](#interactive)
* [用户名-密码流（编程式）](#programmatic)

如果出现以下情况，必须使用授权代码流来获取 Azure AD 访问令牌：

* 在 Azure AD 中启用了双重身份验证。
* 在 Azure AD 中启用了联合身份验证。
* 在应用程序注册期间，不会向你授予对已注册的应用程序的同意。

如果你有权使用用户名和密码登录，则可使用用户名-密码流获取 Azure AD 访问令牌。

### <a name="authorization-code-flow-interactive"></a><a id="authorization-code-flow-interactive"> </a><a id="interactive"> </a>授权代码流（交互式）

可通过两个步骤使用授权代码流来获取 Azure AD 访问令牌。

1. 获取授权代码，该代码会启动一个浏览器窗口并请求用户登录。 用户成功登录后，将返回授权代码。
2. 使用授权代码获取访问令牌。 将同时返回刷新令牌，它可用于刷新访问令牌。

#### <a name="get-the-authorization-code"></a>获取授权代码

> [!NOTE]
>
> 必须成功完成此步骤，然后才能继续操作。 如果遇到“权限”问题，请与管理员联系以获得帮助。

| 参数               | 说明                                                                                                                                                                           |
|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 租户 ID               | Azure AD 中的租户 ID。                                                                                                                                                                |
| 客户端 ID               | [在 Azure 门户中配置应用](#configure-an-app-in-azure-portal)中注册的应用程序的 ID。                                                                        |
| 重定向 URI            | 已注册的应用程序中的其中一个重定向 URI（例如 `http://localhost`）。 会向此 URI 发送身份验证响应，并附带授权代码。 |

##### <a name="get-the-authorization-code-using-a-browser"></a>使用浏览器获取授权代码

这是用于获取 Azure AD 访问令牌的交互式方法。

可通过在浏览器中发送 HTTP 请求来请求授权代码。 有关详细信息，请参阅[请求授权代码](/active-directory/develop/v1-protocols-oauth-code#request-an-authorization-code)。 请相应地替换以下 URL 示例中的字段：

```
// Line breaks for legibility

https://login.microsoftonline.com/<tenant>/oauth2/authorize?client_id=<client-id>
&response_type=code
&redirect_uri=<redirect URI in encoded format: e.g., http%3A%2F%2Flocalhost>
&response_mode=query
&resource=2ff814a6-3304-4ab8-85cb-cd0e6f879c1d
&state=<a random number or some encoded info>
```

出现提示时，将 URL 粘贴到浏览器中并登录 Azure。

> [!div class="mx-imgBorder"]
> ![HTTP 请求 URL](../../../../_static/images/aad/http-request-url.png)

成功登录后，授权代码将附加到返回的 URL 中的代码字段。 保存该代码以供将来使用。

> [!div class="mx-imgBorder"]
> ![授权代码 URL](../../../../_static/images/aad/authorization-code-url.png)

##### <a name="get-the-authorization-code-programmatically"></a>以编程方式获取授权代码

也可使用半编程方式获取授权代码。 以下代码片段将打开浏览器供用户登录。 成功登录后，将返回代码。

1. 使用 `pip install adal` 安装 ADAL Python SDK。
2. 使用 Selenium 库打开浏览器：

   ```bash
   pip install selenium
   ```

3. 下载浏览器驱动程序并将可执行文件提取到 `PATH` 中。 此示例使用的是 Chrome 驱动程序。 下载 [Chrome 驱动程序](https://sites.google.com/a/chromium.org/chromedriver/downloads)。
4. 运行以下代码片段来获取授权代码：

   ```python
   from adal import AuthenticationContext
   from selenium import webdriver
   from urlparse import urlparse, parse_qs
   import time

   authority_host_url = "https://login.microsoftonline.com/"
   azure_databricks_resource_id = "2ff814a6-3304-4ab8-85cb-cd0e6f879c1d"

   # Required user input
   user_parameters = {
        "tenant" : "<tenant-id>",
        "client_id" : "<application-id>",
        "redirect_uri" : "<redirect-uri>"
   }

   template_authz_url = ('https://login.chinacloudapi.cn/{}/oauth2/authorize?'+
        'response_type=code&client_id={}&redirect_uri={}&'+
        'state={}&resource={}')
   # the auth_state can be a random number or can encoded some info
   # about the user. It is used for preventing cross-site request
   # forgery attacks
   auth_state = 12345
   # build the URL to request the authorization code
   authorization_url = template_authz_url.format(
               user_parameters['tenant'],
               user_parameters['client_id'],
               user_parameters['redirect_uri'],
               auth_state,
               azure_databricks_resource_id)

   def get_authorization_code():
     # open a browser, here assume we use Chrome
     dr = webdriver.Chrome()
     # load the user login page
     dr.get(authorization_url)
     # wait until the user login or kill the process
     code_received = False
     code = ''
     while(not code_received):
         cur_url = dr.current_url
         if cur_url.startswith(user_parameters['redirect_uri']):
             parsed = urlparse(cur_url)
             query = parse_qs(parsed.query)
             code = query['code'][0]
             state = query['state'][0]
             # throw exception if the state does not match
             if state != str(auth_state):
                 raise ValueError('state does not match')
             code_received = True
             dr.close()

     if not code_received:
         print 'Error in requesting authorization code'
         dr.close()
     # authorization code is returned. If not successful,
     # then an empty code is returned
     return code
   ```

#### <a name="use-the-authorization-code-to-obtain-the-access-and-refresh-tokens"></a><a id="get-token"> </a><a id="use-the-authorization-code-to-obtain-the-access-and-refresh-tokens"> </a>使用授权代码获取访问令牌和刷新令牌

```python
def get_refresh_and_access_token():
  # configure AuthenticationContext
  # authority URL and tenant ID are used
  authority_url = authority_host_url + user_parameters['tenant']
  context = AuthenticationContext(authority_url)

  # Obtain the authorization code in by a HTTP request in the browser
  # then copy it here or, call the function above to get the authorization code
  authz_code = get_authorization_code()

  # API call to get the token, the response is a
  # key-value dict
  token_response = context.acquire_token_with_authorization_code(
    authz_code,
    user_parameters['redirect_uri'],
    azure_databricks_resource_id,
    user_parameters['clientId'])

  # you can print all the fields in the token_response
  for key in token_response.keys():
    print str(key) + ': ' + str(token_response[key])

  # the tokens can be returned as a pair (or you can return the full
  # token_response)
  return (token_response['accessToken'], token_response['refreshToken'])
```

### <a name="username-password-flow-programmatic"></a><a id="programmatic"> </a><a id="username-password-flow-programmatic"> </a>用户名-密码流（编程式）

如果你有权使用用户名和密码登录，可使用此编程方法获取 Azure AD 访问令牌。

| 参数               | 说明                                                                                                                |
|-------------------------|----------------------------------------------------------------------------------------------------------------------------|
| 租户 ID               | Azure AD 中的租户 ID。                                                                                                     |
| 客户端 ID               | [在 Azure 门户中配置应用](#configure-an-app-in-azure-portal)中注册的应用程序的应用程序的 ID。 |
| 使用rname 和“domain2.contoso.com” Password   | 租户中用户的用户名（即登录到 Azure 门户时的电子邮件地址）和密码。           |

可使用以下示例代码通过用户名和密码获取 Azure AD 访问令牌。 省略了错误处理。 有关获取令牌时可能出现的错误的列表，请参阅 ADAL GitHub 存储库中的 [get_token](https://github.com/AzureAD/azure-activedirectory-library-for-python/blob/18a985c7f3f8eb6b5e35bd9df90a08ad8bccb42e/adal/oauth2_client.py#L255) 函数定义。

```python
from adal import AuthenticationContext

authority_host_url = "https://login.microsoftonline.com/"
# the Application ID of  AzureDatabricks
azure_databricks_resource_id = "2ff814a6-3304-4ab8-85cb-cd0e6f879c1d"

# Required user input
user_parameters = {
   "tenant" : "<tenant-id>",
   "client_id" : "<application-id>",
   "username" : "<username>",
   "password" : "<password>"
}

# configure AuthenticationContext
# authority URL and tenant ID are used
authority_url = authority_host_url + user_parameters['tenant']
context = AuthenticationContext(authority_url)

# API call to get the token
token_response = context.acquire_token_with_username_password(
  azure_databricks_resource_id,
  user_parameters['username'],
  user_parameters['password'],
  user_parameters['client_id']
)

access_token = token_response['accessToken']
refresh_token = token_response['refreshToken']
```

## <a name="use-an-azure-ad-access-token-to-access-the-databricks-rest-api"></a><a id="use-an-azure-ad-access-token-to-access-the-databricks-rest-api"> </a><a id="use-token"> </a>使用 Azure AD 访问令牌访问 Databricks REST API

本部分介绍如何使用 Azure AD 令牌调用 Databricks REST API。 在以下示例中，请将 `<databricks-instance>` 替换为 Azure Databricks 部署的[每工作区 URL](../../../../workspace/workspace-details.md#per-workspace-url)。

### <a name="python-example"></a>Python 示例

此示例演示如何列出 Azure Databricks 工作区中的群集。 它使用在[使用授权代码获取访问令牌和刷新令牌](#get-token)中定义的 `get_refresh_and_access_token` 方法来获取令牌。

```python
import requests

(refresh_token, access_token) =  get_refresh_and_access_token()

def list_cluster_with_aad_token():

  domain = '<databricks-instance>'
  token = access_token
  base_url = 'https://%s/api/2.0/clusters/list' % (domain)

  # request header
  headers = {
    'Authorization' : 'Bearer ' + token
  }

  response = requests.get(
    base_url,
    headers=headers
  )

  print 'response header: ' + str(response.headers)
  print 'the response is: ' + str(response.content)

  try:
    print 'Decoding response as JSON... '
    res_json = response.json()

    for cluster in res_json['clusters']:
        print str(cluster)
  except Exception as e:
    print 'Response cannot be parsed as JSON:'
    print '\t: ' + str(response)
    print 'The exception is: %s' % str(e)
```

> [!NOTE]
>
> 如果你不是管理员用户，但想以管理员用户身份登录，则除了 `'Authorization' : 'Bearer '` 标头外，还必须提供 `X-Databricks-Azure-Workspace-Resource-Id` 标头，并且对于 Azure 中的工作区资源，你必须是参与者或所有者角色。 按如下所示构造 `X-Databricks-Azure-Workspace-Resource-Id` 值：
>
> ```python
> subscription = '<azure-subscription-id>'
> resource_group = '<azure-resource-group-name>'
> workspace = '<databricks-workspace-name-in-azure>'
>
> db_resource_id = '/subscriptions/%s/resourcegroups/%s/providers/microsoft.databricks/workspaces/%s' % (
>   subscription,
>   resource_group,
>   workspace
> )
>
> headers = {
>   'Authorization' : 'Bearer ' + access_token,
>   'X-Databricks-Azure-Workspace-Resource-Id' : db_resource_id
> }
> ```

### <a name="curl-example"></a>`curl` 实例

```bash
curl -X GET \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer <access-token>' \
https://<databricks-instance>/api/2.0/clusters/list
```

## <a name="refresh-an-access-token"></a>刷新访问令牌

如果你获取了刷新令牌和访问令牌，可使用刷新令牌来获取新的令牌。
访问令牌的生存期默认为 1 小时。 可使用 [Azure Active Directory 中的可配置令牌生存期](/active-directory/develop/active-directory-configurable-token-lifetimes)中的方法配置访问令牌的生存期。

```python
# supply the refresh_token (whose default lifetime is 90 days or longer [token lifetime])
def refresh_access_token(refresh_token):
  context = AuthenticationContext(authority_url)
  # function link
  token_response = context.acquire_token_with_refresh_token(
                  refresh_token,
                  user_parameters['client_id'],
                  azure_databricks_resource_id)
  # print all the fields in the token_response
  for key in token_response.keys():
      print str(key) + ': ' + str(token_response[key])

  # the new 'refreshToken' and  'accessToken' will be returned
  return (token_response['refreshToken'], token_response['accessToken'])
```