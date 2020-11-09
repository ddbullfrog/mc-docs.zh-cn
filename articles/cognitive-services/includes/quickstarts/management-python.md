---
title: 快速入门：适用于 Python 的 Azure 管理客户端库
description: 在本快速入门中，开始使用适用于 Python 的 Azure 管理客户端库。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 10/28/2020
ms.author: v-johya
ms.openlocfilehash: d9b36657b33d671ec5099f315ce81eadba60b9ef
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106478"
---
[参考文档](https://docs.microsoft.com/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices?view=azure-python) | [库源代码](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-mgmt-cognitiveservices) | [包 (PyPi)](https://pypi.org/project/azure-mgmt-cognitiveservices/) | [示例](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-mgmt-cognitiveservices/tests)

## <a name="prerequisites"></a>先决条件

* 有效的 Azure 订阅 - [创建试用订阅](https://www.azure.cn/pricing/1rmb-trial/)。
* [Python 3.x](https://www.python.org/)

[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-python-application"></a>创建新的 Python 应用程序

在首选编辑器或 IDE 中创建新的 Python 应用程序，并在控制台窗口中导航到你的项目。

### <a name="install-the-client-library"></a>安装客户端库

可使用以下方式安装客户端库：

```console
pip install azure-mgmt-cognitiveservices
```

如果你使用的是 Visual Studio IDE，客户端库可用作可下载的 NuGet 包。

### <a name="import-libraries"></a>导入库

打开 Python 脚本并导入以下库。

```python
# <snippet_imports>
from msrestazure.azure_active_directory import ServicePrincipalCredentials
from azure.mgmt.cognitiveservices import CognitiveServicesManagementClient
from azure.mgmt.cognitiveservices.models import CognitiveServicesAccount, Sku
# </snippet_imports>

# Azure Management
#
# This script requires the following modules:
#   python -m pip install azure-mgmt-cognitiveservices
#   python -m pip install msrestazure
#
# SDK: https://docs.microsoft.com/en-us/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices?view=azure-python 
#
# This script runs under Python 3.4 or later.
# The application ID and secret of the service principal you are using to connect to the Azure Management Service.

# To create a service principal with the Azure CLI, see:
# /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
# To install the Azure CLI, see:
# /cli/install-azure-cli?view=azure-cli-latest

# To create a service principal with Azure PowerShell, see: 
# https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
# To install Azure PowerShell, see:
# https://github.com/Azure/azure-powershell

# When you create a service principal, you will see it has both an ID and an application ID.
# For example, if you create a service principal with Azure PowerShell, it should look like the following:

# Secret                : System.Security.SecureString
# ServicePrincipalNames : {<application ID>, <application URL>}
# ApplicationId         : <application ID>
# ObjectType            : ServicePrincipal
# DisplayName           : <name>
# Id                    : <ID>
# Type                  :

# <snippet_constants>
# Be sure to use the service pricipal application ID, not simply the ID. 
service_principal_application_id = "MY-SERVICE-PRINCIPAL-APPLICATION-ID"
service_principal_secret = "MY-SERVICE-PRINCIPAL-SECRET"

# The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions.
subscription_id = "MY-SUBSCRIPTION-ID"

# The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory.
tenant_id = "MY-TENANT-ID"

# The name of the Azure resource group in which you want to create the resource.
# You can find resource groups in the Azure Dashboard under Home > Resource groups.
resource_group_name = "MY-RESOURCE-GROUP"
# </snippet_constants>

# <snippet_auth>
credentials = ServicePrincipalCredentials(service_principal_application_id, service_principal_secret, tenant=tenant_id)
client = CognitiveServicesManagementClient(credentials, subscription_id)
# </snippet_auth>

# <snippet_list_avail>
def list_available_kinds_skus_locations():
    print("Available SKUs:")
    result = client.resource_skus.list()
    print("Kind\tSKU Name\tSKU Tier\tLocations")
    for x in result:
        locations = ",".join(x.locations)
        print(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations)
# </snippet_list_avail>

# Note Azure resources are also sometimes referred to as accounts.

# <snippet_list>
def list_resources():
    print("Resources in resource group: " + resource_group_name)
    result = client.accounts.list_by_resource_group(resource_group_name)
    for x in result:
        print(x)
        print()
# </snippet_list>

# <snippet_create>
def create_resource (resource_name, kind, sku_name, location):
    print("Creating resource: " + resource_name + "...")
# The parameter "properties" must be an empty object.
    parameters = CognitiveServicesAccount(sku=Sku(name=sku_name), kind=kind, location=location, properties={})
    result = client.accounts.create(resource_group_name, resource_name, parameters)
    print("Resource created.")
    print()
    print("ID: " + result.id)
    print("Name: " + result.name)
    print("Type: " + result.type)
    print()
# </snippet_create>

# <snippet_delete>
def delete_resource(resource_name) :
    print("Deleting resource: " + resource_name + "...")
    client.accounts.delete(resource_group_name, resource_name)
    print("Resource deleted.")
# </snippet_delete>

# <snippet_calls>
# Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
list_available_kinds_skus_locations()

# Create a resource with kind Text Translation, SKU F0 (free tier), location global.
create_resource("test_resource", "TextTranslation", "F0", "Global")

# Uncomment this to list all resources for your Azure account.
list_resources()

# Delete the resource.
delete_resource("test_resource")
# </snippet_calls>
```

## <a name="authenticate-the-client"></a>验证客户端

将以下字段添加到脚本的根目录中，并使用所创建的服务主体和 Azure 帐户信息填写其值。

```python
# <snippet_imports>
from msrestazure.azure_active_directory import ServicePrincipalCredentials
from azure.mgmt.cognitiveservices import CognitiveServicesManagementClient
from azure.mgmt.cognitiveservices.models import CognitiveServicesAccount, Sku
# </snippet_imports>

# Azure Management
#
# This script requires the following modules:
#   python -m pip install azure-mgmt-cognitiveservices
#   python -m pip install msrestazure
#
# SDK: https://docs.microsoft.com/en-us/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices?view=azure-python 
#
# This script runs under Python 3.4 or later.
# The application ID and secret of the service principal you are using to connect to the Azure Management Service.

# To create a service principal with the Azure CLI, see:
# /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
# To install the Azure CLI, see:
# /cli/install-azure-cli?view=azure-cli-latest

# To create a service principal with Azure PowerShell, see: 
# https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
# To install Azure PowerShell, see:
# https://github.com/Azure/azure-powershell

# When you create a service principal, you will see it has both an ID and an application ID.
# For example, if you create a service principal with Azure PowerShell, it should look like the following:

# Secret                : System.Security.SecureString
# ServicePrincipalNames : {<application ID>, <application URL>}
# ApplicationId         : <application ID>
# ObjectType            : ServicePrincipal
# DisplayName           : <name>
# Id                    : <ID>
# Type                  :

# <snippet_constants>
# Be sure to use the service pricipal application ID, not simply the ID. 
service_principal_application_id = "MY-SERVICE-PRINCIPAL-APPLICATION-ID"
service_principal_secret = "MY-SERVICE-PRINCIPAL-SECRET"

# The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions.
subscription_id = "MY-SUBSCRIPTION-ID"

# The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory.
tenant_id = "MY-TENANT-ID"

# The name of the Azure resource group in which you want to create the resource.
# You can find resource groups in the Azure Dashboard under Home > Resource groups.
resource_group_name = "MY-RESOURCE-GROUP"
# </snippet_constants>

# <snippet_auth>
credentials = ServicePrincipalCredentials(service_principal_application_id, service_principal_secret, tenant=tenant_id)
client = CognitiveServicesManagementClient(credentials, subscription_id)
# </snippet_auth>

# <snippet_list_avail>
def list_available_kinds_skus_locations():
    print("Available SKUs:")
    result = client.resource_skus.list()
    print("Kind\tSKU Name\tSKU Tier\tLocations")
    for x in result:
        locations = ",".join(x.locations)
        print(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations)
# </snippet_list_avail>

# Note Azure resources are also sometimes referred to as accounts.

# <snippet_list>
def list_resources():
    print("Resources in resource group: " + resource_group_name)
    result = client.accounts.list_by_resource_group(resource_group_name)
    for x in result:
        print(x)
        print()
# </snippet_list>

# <snippet_create>
def create_resource (resource_name, kind, sku_name, location):
    print("Creating resource: " + resource_name + "...")
# The parameter "properties" must be an empty object.
    parameters = CognitiveServicesAccount(sku=Sku(name=sku_name), kind=kind, location=location, properties={})
    result = client.accounts.create(resource_group_name, resource_name, parameters)
    print("Resource created.")
    print()
    print("ID: " + result.id)
    print("Name: " + result.name)
    print("Type: " + result.type)
    print()
# </snippet_create>

# <snippet_delete>
def delete_resource(resource_name) :
    print("Deleting resource: " + resource_name + "...")
    client.accounts.delete(resource_group_name, resource_name)
    print("Resource deleted.")
# </snippet_delete>

# <snippet_calls>
# Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
list_available_kinds_skus_locations()

# Create a resource with kind Text Translation, SKU F0 (free tier), location global.
create_resource("test_resource", "TextTranslation", "F0", "Global")

# Uncomment this to list all resources for your Azure account.
list_resources()

# Delete the resource.
delete_resource("test_resource")
# </snippet_calls>
```

然后添加以下代码来构造 CognitiveServicesManagementClient 对象。 所有 Azure 管理操作都需要此对象。

```python
# <snippet_imports>
from msrestazure.azure_active_directory import ServicePrincipalCredentials
from azure.mgmt.cognitiveservices import CognitiveServicesManagementClient
from azure.mgmt.cognitiveservices.models import CognitiveServicesAccount, Sku
# </snippet_imports>

# Azure Management
#
# This script requires the following modules:
#   python -m pip install azure-mgmt-cognitiveservices
#   python -m pip install msrestazure
#
# SDK: https://docs.microsoft.com/en-us/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices?view=azure-python 
#
# This script runs under Python 3.4 or later.
# The application ID and secret of the service principal you are using to connect to the Azure Management Service.

# To create a service principal with the Azure CLI, see:
# /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
# To install the Azure CLI, see:
# /cli/install-azure-cli?view=azure-cli-latest

# To create a service principal with Azure PowerShell, see: 
# https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
# To install Azure PowerShell, see:
# https://github.com/Azure/azure-powershell

# When you create a service principal, you will see it has both an ID and an application ID.
# For example, if you create a service principal with Azure PowerShell, it should look like the following:

# Secret                : System.Security.SecureString
# ServicePrincipalNames : {<application ID>, <application URL>}
# ApplicationId         : <application ID>
# ObjectType            : ServicePrincipal
# DisplayName           : <name>
# Id                    : <ID>
# Type                  :

# <snippet_constants>
# Be sure to use the service pricipal application ID, not simply the ID. 
service_principal_application_id = "MY-SERVICE-PRINCIPAL-APPLICATION-ID"
service_principal_secret = "MY-SERVICE-PRINCIPAL-SECRET"

# The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions.
subscription_id = "MY-SUBSCRIPTION-ID"

# The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory.
tenant_id = "MY-TENANT-ID"

# The name of the Azure resource group in which you want to create the resource.
# You can find resource groups in the Azure Dashboard under Home > Resource groups.
resource_group_name = "MY-RESOURCE-GROUP"
# </snippet_constants>

# <snippet_auth>
credentials = ServicePrincipalCredentials(service_principal_application_id, service_principal_secret, tenant=tenant_id)
client = CognitiveServicesManagementClient(credentials, subscription_id)
# </snippet_auth>

# <snippet_list_avail>
def list_available_kinds_skus_locations():
    print("Available SKUs:")
    result = client.resource_skus.list()
    print("Kind\tSKU Name\tSKU Tier\tLocations")
    for x in result:
        locations = ",".join(x.locations)
        print(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations)
# </snippet_list_avail>

# Note Azure resources are also sometimes referred to as accounts.

# <snippet_list>
def list_resources():
    print("Resources in resource group: " + resource_group_name)
    result = client.accounts.list_by_resource_group(resource_group_name)
    for x in result:
        print(x)
        print()
# </snippet_list>

# <snippet_create>
def create_resource (resource_name, kind, sku_name, location):
    print("Creating resource: " + resource_name + "...")
# The parameter "properties" must be an empty object.
    parameters = CognitiveServicesAccount(sku=Sku(name=sku_name), kind=kind, location=location, properties={})
    result = client.accounts.create(resource_group_name, resource_name, parameters)
    print("Resource created.")
    print()
    print("ID: " + result.id)
    print("Name: " + result.name)
    print("Type: " + result.type)
    print()
# </snippet_create>

# <snippet_delete>
def delete_resource(resource_name) :
    print("Deleting resource: " + resource_name + "...")
    client.accounts.delete(resource_group_name, resource_name)
    print("Resource deleted.")
# </snippet_delete>

# <snippet_calls>
# Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
list_available_kinds_skus_locations()

# Create a resource with kind Text Translation, SKU F0 (free tier), location global.
create_resource("test_resource", "TextTranslation", "F0", "Global")

# Uncomment this to list all resources for your Azure account.
list_resources()

# Delete the resource.
delete_resource("test_resource")
# </snippet_calls>
```

## <a name="create-a-cognitive-services-resource"></a>创建认知服务资源

### <a name="choose-a-service-and-pricing-tier"></a>选择服务和定价层

创建新资源时，需要知道要使用的服务的种类，以及所需的[定价层](https://www.azure.cn/pricing/details/cognitive-services/)（或 SKU）。 创建资源时，将此信息和其他信息用作参数。 以下函数列出了可用的认知服务种类。

```python
# <snippet_imports>
from msrestazure.azure_active_directory import ServicePrincipalCredentials
from azure.mgmt.cognitiveservices import CognitiveServicesManagementClient
from azure.mgmt.cognitiveservices.models import CognitiveServicesAccount, Sku
# </snippet_imports>

# Azure Management
#
# This script requires the following modules:
#   python -m pip install azure-mgmt-cognitiveservices
#   python -m pip install msrestazure
#
# SDK: https://docs.microsoft.com/en-us/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices?view=azure-python 
#
# This script runs under Python 3.4 or later.
# The application ID and secret of the service principal you are using to connect to the Azure Management Service.

# To create a service principal with the Azure CLI, see:
# /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
# To install the Azure CLI, see:
# /cli/install-azure-cli?view=azure-cli-latest

# To create a service principal with Azure PowerShell, see: 
# https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
# To install Azure PowerShell, see:
# https://github.com/Azure/azure-powershell

# When you create a service principal, you will see it has both an ID and an application ID.
# For example, if you create a service principal with Azure PowerShell, it should look like the following:

# Secret                : System.Security.SecureString
# ServicePrincipalNames : {<application ID>, <application URL>}
# ApplicationId         : <application ID>
# ObjectType            : ServicePrincipal
# DisplayName           : <name>
# Id                    : <ID>
# Type                  :

# <snippet_constants>
# Be sure to use the service pricipal application ID, not simply the ID. 
service_principal_application_id = "MY-SERVICE-PRINCIPAL-APPLICATION-ID"
service_principal_secret = "MY-SERVICE-PRINCIPAL-SECRET"

# The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions.
subscription_id = "MY-SUBSCRIPTION-ID"

# The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory.
tenant_id = "MY-TENANT-ID"

# The name of the Azure resource group in which you want to create the resource.
# You can find resource groups in the Azure Dashboard under Home > Resource groups.
resource_group_name = "MY-RESOURCE-GROUP"
# </snippet_constants>

# <snippet_auth>
credentials = ServicePrincipalCredentials(service_principal_application_id, service_principal_secret, tenant=tenant_id)
client = CognitiveServicesManagementClient(credentials, subscription_id)
# </snippet_auth>

# <snippet_list_avail>
def list_available_kinds_skus_locations():
    print("Available SKUs:")
    result = client.resource_skus.list()
    print("Kind\tSKU Name\tSKU Tier\tLocations")
    for x in result:
        locations = ",".join(x.locations)
        print(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations)
# </snippet_list_avail>

# Note Azure resources are also sometimes referred to as accounts.

# <snippet_list>
def list_resources():
    print("Resources in resource group: " + resource_group_name)
    result = client.accounts.list_by_resource_group(resource_group_name)
    for x in result:
        print(x)
        print()
# </snippet_list>

# <snippet_create>
def create_resource (resource_name, kind, sku_name, location):
    print("Creating resource: " + resource_name + "...")
# The parameter "properties" must be an empty object.
    parameters = CognitiveServicesAccount(sku=Sku(name=sku_name), kind=kind, location=location, properties={})
    result = client.accounts.create(resource_group_name, resource_name, parameters)
    print("Resource created.")
    print()
    print("ID: " + result.id)
    print("Name: " + result.name)
    print("Type: " + result.type)
    print()
# </snippet_create>

# <snippet_delete>
def delete_resource(resource_name) :
    print("Deleting resource: " + resource_name + "...")
    client.accounts.delete(resource_group_name, resource_name)
    print("Resource deleted.")
# </snippet_delete>

# <snippet_calls>
# Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
list_available_kinds_skus_locations()

# Create a resource with kind Text Translation, SKU F0 (free tier), location global.
create_resource("test_resource", "TextTranslation", "F0", "Global")

# Uncomment this to list all resources for your Azure account.
list_resources()

# Delete the resource.
delete_resource("test_resource")
# </snippet_calls>
```

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="create-a-cognitive-services-resource"></a>创建认知服务资源

若要创建并订阅新的认知服务资源，请使用 Create 函数。 此函数向传入的资源组添加新的可计费资源。 创建新资源时，需要知道要使用的服务的种类，以及其定价层（或 SKU）和 Azure 位置。 下面的函数使用所有这些参数并创建资源。

```python
# <snippet_imports>
from msrestazure.azure_active_directory import ServicePrincipalCredentials
from azure.mgmt.cognitiveservices import CognitiveServicesManagementClient
from azure.mgmt.cognitiveservices.models import CognitiveServicesAccount, Sku
# </snippet_imports>

# Azure Management
#
# This script requires the following modules:
#   python -m pip install azure-mgmt-cognitiveservices
#   python -m pip install msrestazure
#
# SDK: https://docs.microsoft.com/en-us/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices?view=azure-python 
#
# This script runs under Python 3.4 or later.
# The application ID and secret of the service principal you are using to connect to the Azure Management Service.

# To create a service principal with the Azure CLI, see:
# /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
# To install the Azure CLI, see:
# /cli/install-azure-cli?view=azure-cli-latest

# To create a service principal with Azure PowerShell, see: 
# https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
# To install Azure PowerShell, see:
# https://github.com/Azure/azure-powershell

# When you create a service principal, you will see it has both an ID and an application ID.
# For example, if you create a service principal with Azure PowerShell, it should look like the following:

# Secret                : System.Security.SecureString
# ServicePrincipalNames : {<application ID>, <application URL>}
# ApplicationId         : <application ID>
# ObjectType            : ServicePrincipal
# DisplayName           : <name>
# Id                    : <ID>
# Type                  :

# <snippet_constants>
# Be sure to use the service pricipal application ID, not simply the ID. 
service_principal_application_id = "MY-SERVICE-PRINCIPAL-APPLICATION-ID"
service_principal_secret = "MY-SERVICE-PRINCIPAL-SECRET"

# The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions.
subscription_id = "MY-SUBSCRIPTION-ID"

# The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory.
tenant_id = "MY-TENANT-ID"

# The name of the Azure resource group in which you want to create the resource.
# You can find resource groups in the Azure Dashboard under Home > Resource groups.
resource_group_name = "MY-RESOURCE-GROUP"
# </snippet_constants>

# <snippet_auth>
credentials = ServicePrincipalCredentials(service_principal_application_id, service_principal_secret, tenant=tenant_id)
client = CognitiveServicesManagementClient(credentials, subscription_id)
# </snippet_auth>

# <snippet_list_avail>
def list_available_kinds_skus_locations():
    print("Available SKUs:")
    result = client.resource_skus.list()
    print("Kind\tSKU Name\tSKU Tier\tLocations")
    for x in result:
        locations = ",".join(x.locations)
        print(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations)
# </snippet_list_avail>

# Note Azure resources are also sometimes referred to as accounts.

# <snippet_list>
def list_resources():
    print("Resources in resource group: " + resource_group_name)
    result = client.accounts.list_by_resource_group(resource_group_name)
    for x in result:
        print(x)
        print()
# </snippet_list>

# <snippet_create>
def create_resource (resource_name, kind, sku_name, location):
    print("Creating resource: " + resource_name + "...")
# The parameter "properties" must be an empty object.
    parameters = CognitiveServicesAccount(sku=Sku(name=sku_name), kind=kind, location=location, properties={})
    result = client.accounts.create(resource_group_name, resource_name, parameters)
    print("Resource created.")
    print()
    print("ID: " + result.id)
    print("Name: " + result.name)
    print("Type: " + result.type)
    print()
# </snippet_create>

# <snippet_delete>
def delete_resource(resource_name) :
    print("Deleting resource: " + resource_name + "...")
    client.accounts.delete(resource_group_name, resource_name)
    print("Resource deleted.")
# </snippet_delete>

# <snippet_calls>
# Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
list_available_kinds_skus_locations()

# Create a resource with kind Text Translation, SKU F0 (free tier), location global.
create_resource("test_resource", "TextTranslation", "F0", "Global")

# Uncomment this to list all resources for your Azure account.
list_resources()

# Delete the resource.
delete_resource("test_resource")
# </snippet_calls>
```

## <a name="view-your-resources"></a>查看资源

若要查看 Azure 帐户下的所有资源（跨所有资源组），请使用以下函数：

```python
# <snippet_imports>
from msrestazure.azure_active_directory import ServicePrincipalCredentials
from azure.mgmt.cognitiveservices import CognitiveServicesManagementClient
from azure.mgmt.cognitiveservices.models import CognitiveServicesAccount, Sku
# </snippet_imports>

# Azure Management
#
# This script requires the following modules:
#   python -m pip install azure-mgmt-cognitiveservices
#   python -m pip install msrestazure
#
# SDK: https://docs.microsoft.com/en-us/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices?view=azure-python 
#
# This script runs under Python 3.4 or later.
# The application ID and secret of the service principal you are using to connect to the Azure Management Service.

# To create a service principal with the Azure CLI, see:
# /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
# To install the Azure CLI, see:
# /cli/install-azure-cli?view=azure-cli-latest

# To create a service principal with Azure PowerShell, see: 
# https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
# To install Azure PowerShell, see:
# https://github.com/Azure/azure-powershell

# When you create a service principal, you will see it has both an ID and an application ID.
# For example, if you create a service principal with Azure PowerShell, it should look like the following:

# Secret                : System.Security.SecureString
# ServicePrincipalNames : {<application ID>, <application URL>}
# ApplicationId         : <application ID>
# ObjectType            : ServicePrincipal
# DisplayName           : <name>
# Id                    : <ID>
# Type                  :

# <snippet_constants>
# Be sure to use the service pricipal application ID, not simply the ID. 
service_principal_application_id = "MY-SERVICE-PRINCIPAL-APPLICATION-ID"
service_principal_secret = "MY-SERVICE-PRINCIPAL-SECRET"

# The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions.
subscription_id = "MY-SUBSCRIPTION-ID"

# The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory.
tenant_id = "MY-TENANT-ID"

# The name of the Azure resource group in which you want to create the resource.
# You can find resource groups in the Azure Dashboard under Home > Resource groups.
resource_group_name = "MY-RESOURCE-GROUP"
# </snippet_constants>

# <snippet_auth>
credentials = ServicePrincipalCredentials(service_principal_application_id, service_principal_secret, tenant=tenant_id)
client = CognitiveServicesManagementClient(credentials, subscription_id)
# </snippet_auth>

# <snippet_list_avail>
def list_available_kinds_skus_locations():
    print("Available SKUs:")
    result = client.resource_skus.list()
    print("Kind\tSKU Name\tSKU Tier\tLocations")
    for x in result:
        locations = ",".join(x.locations)
        print(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations)
# </snippet_list_avail>

# Note Azure resources are also sometimes referred to as accounts.

# <snippet_list>
def list_resources():
    print("Resources in resource group: " + resource_group_name)
    result = client.accounts.list_by_resource_group(resource_group_name)
    for x in result:
        print(x)
        print()
# </snippet_list>

# <snippet_create>
def create_resource (resource_name, kind, sku_name, location):
    print("Creating resource: " + resource_name + "...")
# The parameter "properties" must be an empty object.
    parameters = CognitiveServicesAccount(sku=Sku(name=sku_name), kind=kind, location=location, properties={})
    result = client.accounts.create(resource_group_name, resource_name, parameters)
    print("Resource created.")
    print()
    print("ID: " + result.id)
    print("Name: " + result.name)
    print("Type: " + result.type)
    print()
# </snippet_create>

# <snippet_delete>
def delete_resource(resource_name) :
    print("Deleting resource: " + resource_name + "...")
    client.accounts.delete(resource_group_name, resource_name)
    print("Resource deleted.")
# </snippet_delete>

# <snippet_calls>
# Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
list_available_kinds_skus_locations()

# Create a resource with kind Text Translation, SKU F0 (free tier), location global.
create_resource("test_resource", "TextTranslation", "F0", "Global")

# Uncomment this to list all resources for your Azure account.
list_resources()

# Delete the resource.
delete_resource("test_resource")
# </snippet_calls>
```

## <a name="delete-a-resource"></a>删除资源

下面的函数从给定的资源组中删除指定的资源。

```python
# <snippet_imports>
from msrestazure.azure_active_directory import ServicePrincipalCredentials
from azure.mgmt.cognitiveservices import CognitiveServicesManagementClient
from azure.mgmt.cognitiveservices.models import CognitiveServicesAccount, Sku
# </snippet_imports>

# Azure Management
#
# This script requires the following modules:
#   python -m pip install azure-mgmt-cognitiveservices
#   python -m pip install msrestazure
#
# SDK: https://docs.microsoft.com/en-us/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices?view=azure-python 
#
# This script runs under Python 3.4 or later.
# The application ID and secret of the service principal you are using to connect to the Azure Management Service.

# To create a service principal with the Azure CLI, see:
# /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
# To install the Azure CLI, see:
# /cli/install-azure-cli?view=azure-cli-latest

# To create a service principal with Azure PowerShell, see: 
# https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
# To install Azure PowerShell, see:
# https://github.com/Azure/azure-powershell

# When you create a service principal, you will see it has both an ID and an application ID.
# For example, if you create a service principal with Azure PowerShell, it should look like the following:

# Secret                : System.Security.SecureString
# ServicePrincipalNames : {<application ID>, <application URL>}
# ApplicationId         : <application ID>
# ObjectType            : ServicePrincipal
# DisplayName           : <name>
# Id                    : <ID>
# Type                  :

# <snippet_constants>
# Be sure to use the service pricipal application ID, not simply the ID. 
service_principal_application_id = "MY-SERVICE-PRINCIPAL-APPLICATION-ID"
service_principal_secret = "MY-SERVICE-PRINCIPAL-SECRET"

# The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions.
subscription_id = "MY-SUBSCRIPTION-ID"

# The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory.
tenant_id = "MY-TENANT-ID"

# The name of the Azure resource group in which you want to create the resource.
# You can find resource groups in the Azure Dashboard under Home > Resource groups.
resource_group_name = "MY-RESOURCE-GROUP"
# </snippet_constants>

# <snippet_auth>
credentials = ServicePrincipalCredentials(service_principal_application_id, service_principal_secret, tenant=tenant_id)
client = CognitiveServicesManagementClient(credentials, subscription_id)
# </snippet_auth>

# <snippet_list_avail>
def list_available_kinds_skus_locations():
    print("Available SKUs:")
    result = client.resource_skus.list()
    print("Kind\tSKU Name\tSKU Tier\tLocations")
    for x in result:
        locations = ",".join(x.locations)
        print(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations)
# </snippet_list_avail>

# Note Azure resources are also sometimes referred to as accounts.

# <snippet_list>
def list_resources():
    print("Resources in resource group: " + resource_group_name)
    result = client.accounts.list_by_resource_group(resource_group_name)
    for x in result:
        print(x)
        print()
# </snippet_list>

# <snippet_create>
def create_resource (resource_name, kind, sku_name, location):
    print("Creating resource: " + resource_name + "...")
# The parameter "properties" must be an empty object.
    parameters = CognitiveServicesAccount(sku=Sku(name=sku_name), kind=kind, location=location, properties={})
    result = client.accounts.create(resource_group_name, resource_name, parameters)
    print("Resource created.")
    print()
    print("ID: " + result.id)
    print("Name: " + result.name)
    print("Type: " + result.type)
    print()
# </snippet_create>

# <snippet_delete>
def delete_resource(resource_name) :
    print("Deleting resource: " + resource_name + "...")
    client.accounts.delete(resource_group_name, resource_name)
    print("Resource deleted.")
# </snippet_delete>

# <snippet_calls>
# Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
list_available_kinds_skus_locations()

# Create a resource with kind Text Translation, SKU F0 (free tier), location global.
create_resource("test_resource", "TextTranslation", "F0", "Global")

# Uncomment this to list all resources for your Azure account.
list_resources()

# Delete the resource.
delete_resource("test_resource")
# </snippet_calls>
```

## <a name="call-management-functions"></a>调用管理函数

将以下代码添加到脚本底部，以调用上述函数。 此代码列出可用资源、创建示例资源、列出拥有的资源，然后删除示例资源。

```python
# <snippet_imports>
from msrestazure.azure_active_directory import ServicePrincipalCredentials
from azure.mgmt.cognitiveservices import CognitiveServicesManagementClient
from azure.mgmt.cognitiveservices.models import CognitiveServicesAccount, Sku
# </snippet_imports>

# Azure Management
#
# This script requires the following modules:
#   python -m pip install azure-mgmt-cognitiveservices
#   python -m pip install msrestazure
#
# SDK: https://docs.microsoft.com/en-us/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices?view=azure-python 
#
# This script runs under Python 3.4 or later.
# The application ID and secret of the service principal you are using to connect to the Azure Management Service.

# To create a service principal with the Azure CLI, see:
# /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
# To install the Azure CLI, see:
# /cli/install-azure-cli?view=azure-cli-latest

# To create a service principal with Azure PowerShell, see: 
# https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
# To install Azure PowerShell, see:
# https://github.com/Azure/azure-powershell

# When you create a service principal, you will see it has both an ID and an application ID.
# For example, if you create a service principal with Azure PowerShell, it should look like the following:

# Secret                : System.Security.SecureString
# ServicePrincipalNames : {<application ID>, <application URL>}
# ApplicationId         : <application ID>
# ObjectType            : ServicePrincipal
# DisplayName           : <name>
# Id                    : <ID>
# Type                  :

# <snippet_constants>
# Be sure to use the service pricipal application ID, not simply the ID. 
service_principal_application_id = "MY-SERVICE-PRINCIPAL-APPLICATION-ID"
service_principal_secret = "MY-SERVICE-PRINCIPAL-SECRET"

# The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions.
subscription_id = "MY-SUBSCRIPTION-ID"

# The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory.
tenant_id = "MY-TENANT-ID"

# The name of the Azure resource group in which you want to create the resource.
# You can find resource groups in the Azure Dashboard under Home > Resource groups.
resource_group_name = "MY-RESOURCE-GROUP"
# </snippet_constants>

# <snippet_auth>
credentials = ServicePrincipalCredentials(service_principal_application_id, service_principal_secret, tenant=tenant_id)
client = CognitiveServicesManagementClient(credentials, subscription_id)
# </snippet_auth>

# <snippet_list_avail>
def list_available_kinds_skus_locations():
    print("Available SKUs:")
    result = client.resource_skus.list()
    print("Kind\tSKU Name\tSKU Tier\tLocations")
    for x in result:
        locations = ",".join(x.locations)
        print(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations)
# </snippet_list_avail>

# Note Azure resources are also sometimes referred to as accounts.

# <snippet_list>
def list_resources():
    print("Resources in resource group: " + resource_group_name)
    result = client.accounts.list_by_resource_group(resource_group_name)
    for x in result:
        print(x)
        print()
# </snippet_list>

# <snippet_create>
def create_resource (resource_name, kind, sku_name, location):
    print("Creating resource: " + resource_name + "...")
# The parameter "properties" must be an empty object.
    parameters = CognitiveServicesAccount(sku=Sku(name=sku_name), kind=kind, location=location, properties={})
    result = client.accounts.create(resource_group_name, resource_name, parameters)
    print("Resource created.")
    print()
    print("ID: " + result.id)
    print("Name: " + result.name)
    print("Type: " + result.type)
    print()
# </snippet_create>

# <snippet_delete>
def delete_resource(resource_name) :
    print("Deleting resource: " + resource_name + "...")
    client.accounts.delete(resource_group_name, resource_name)
    print("Resource deleted.")
# </snippet_delete>

# <snippet_calls>
# Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
list_available_kinds_skus_locations()

# Create a resource with kind Text Translation, SKU F0 (free tier), location global.
create_resource("test_resource", "TextTranslation", "F0", "Global")

# Uncomment this to list all resources for your Azure account.
list_resources()

# Delete the resource.
delete_resource("test_resource")
# </snippet_calls>
```

## <a name="run-the-application"></a>运行应用程序

在命令行中使用 `python` 命令运行应用程序。

```console
python <your-script-name>.py
```

## <a name="see-also"></a>另请参阅

* [Azure 管理 SDK 参考文档](https://docs.microsoft.com/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices?view=azure-python)
* [什么是 Azure 认知服务？](../../Welcome.md)
* [对 Azure 认知服务的请求进行身份验证](../../authentication.md)
* [使用 Azure 门户创建新资源](../../cognitive-services-apis-create-account.md)

