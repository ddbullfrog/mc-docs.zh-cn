---
title: 快速入门：适用于 Node.js 的 Azure 管理客户端库
description: 在本快速入门中，开始使用适用于 Node.js 的 Azure 管理客户端库。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 10/28/2020
ms.author: v-johya
ms.openlocfilehash: e7315683eee3fb5d1021363ec9e85aa884d70caf
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106473"
---
[参考文档](https://docs.microsoft.com/javascript/api/@azure/arm-cognitiveservices/?view=azure-node-latest) | [库源代码](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/arm-cognitiveservices) | [包 (NPM)](https://www.npmjs.com/package/@azure/arm-cognitiveservices) | [示例](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/arm-cognitiveservices#sample-code)

## <a name="prerequisites"></a>先决条件

* 有效的 Azure 订阅 - [创建试用订阅](https://www.azure.cn/pricing/1rmb-trial/)。
* 最新版本的 [Node.js](https://nodejs.org/)

[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-nodejs-application"></a>创建新的 Node.js 应用程序

在控制台窗口（例如 cmd、PowerShell 或 Bash）中，为应用创建一个新目录并导航到该目录。 

```console
mkdir myapp && cd myapp
```

运行 `npm init` 命令以使用 `package.json` 文件创建一个 node 应用程序。 

```console
npm init
```

继续之前，请先创建一个名为 index.js 的文件。

### <a name="install-the-client-library"></a>安装客户端库

安装以下 NPM 包：

```console
npm install @azure/arm-cognitiveservices
npm install @azure/ms-rest-js
npm install @azure/ms-rest-nodeauth
```

应用的 `package.json` 文件将使用依赖项进行更新。

### <a name="import-libraries"></a>导入库

打开 index.js 脚本并导入以下库。

```javascript
// <snippet_imports>
"use strict";
/* To run this sample, install the following modules.
 * npm install @azure/arm-cognitiveservices
 * npm install @azure/ms-rest-js
 * npm install @azure/ms-rest-nodeauth
 */
var Arm = require("@azure/arm-cognitiveservices");
var msRestNodeAuth = require("@azure/ms-rest-nodeauth");
// </snippet_imports>

/*
The application ID and secret of the service principal you are using to connect to the Azure Management Service.

To create a service principal with the Azure CLI, see:
/cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
To install the Azure CLI, see:
/cli/install-azure-cli?view=azure-cli-latest

To create a service principal with Azure PowerShell, see: 
https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
To install Azure PowerShell, see:
https://github.com/Azure/azure-powershell

When you create a service principal, you will see it has both an ID and an application ID.
For example, if you create a service principal with Azure PowerShell, it should look like the following:

Secret                : System.Security.SecureString
ServicePrincipalNames : {<application ID>, <application URL>}
ApplicationId         : <application ID>
ObjectType            : ServicePrincipal
DisplayName           : <name>
Id                    : <ID>
Type                  :

Be sure to use the service pricipal application ID, not simply the ID. 
*/

// <snippet_constants>
const service_principal_application_id = "TODO_REPLACE";
const service_principal_secret = "TODO_REPLACE";

/* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
const subscription_id = "TODO_REPLACE";

/* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
const tenant_id = "TODO_REPLACE";

/* The name of the Azure resource group in which you want to create the resource.
You can find resource groups in the Azure Dashboard under Home > Resource groups. */
const resource_group_name = "TODO_REPLACE";
// </snippet_constants>

// <snippet_list_avail>
async function list_available_kinds_skus_locations (client) {
    console.log ("Available SKUs:");
    var result = await client.list ();
    console.log("Kind\tSKU Name\tSKU Tier\tLocations");
    result.forEach (function (x) {
        var locations = x.locations.join(",");
        console.log(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations);
    });
}
// </snippet_list_avail>

// <snippet_list>
async function list_resources (client) {
    console.log ("Resources in resource group: " + resource_group_name);
    var result = await client.listByResourceGroup (resource_group_name);
    result.forEach (function (x) {
        console.log(x);
        console.log();
    });
}
// </snippet_list>

// <snippet_create>
function create_resource (client, resource_name, kind, sku_name, location) {
    console.log ("Creating resource: " + resource_name + "...");
    // The parameter "properties" must be an empty object.
    var parameters = { sku : { name: sku_name }, kind : kind, location : location, properties : {} };

    return client.create(resource_group_name, resource_name, parameters)
        .then((result) => {
        console.log("Resource created.");
        print();
        console.log("ID: " + result.id);
        console.log("Kind: " + result.kind);
        console.log();
        })
        .catch((err) =>{ 
                console.log(err)
        })
}
// </snippet_create>

// <snippet_delete>
async function delete_resource (client, resource_name) {
    console.log ("Deleting resource: " + resource_name + "...");
    await client.deleteMethod (resource_group_name, resource_name)
    console.log ("Resource deleted.");
    console.log ();
}
// </snippet_delete>

// <snippet_main_auth>
async function quickstart() {
    const credentials = await msRestNodeAuth.loginWithServicePrincipalSecret (service_principal_application_id, service_principal_secret, tenant_id);
    const client = new Arm.CognitiveServicesManagementClient (credentials, subscription_id);
    // Note Azure resources are also sometimes referred to as accounts.
    const accounts_client = new Arm.Accounts (client);
    const resource_skus_client = new Arm.ResourceSkus (client);
    // </snippet_main_auth>

    // <snippet_main_calls>
    // Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
    list_available_kinds_skus_locations (resource_skus_client);

    // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
    await create_resource (accounts_client, "test_resource", "TextTranslation", "F0", "Global");

    // Uncomment this to list all resources for your Azure account.
    list_resources (accounts_client);

    // Delete the resource.
    delete_resource (accounts_client, "test_resource");
}
// </snippet_main_calls>

// <snippet_main>
try {
    quickstart();
}
catch (error) {
    console.log(error);
}
// </snippet_main>
```

## <a name="authenticate-the-client"></a>验证客户端

将以下字段添加到脚本的根目录中，并使用所创建的服务主体和 Azure 帐户信息填写其值。

```javascript
// <snippet_imports>
"use strict";
/* To run this sample, install the following modules.
 * npm install @azure/arm-cognitiveservices
 * npm install @azure/ms-rest-js
 * npm install @azure/ms-rest-nodeauth
 */
var Arm = require("@azure/arm-cognitiveservices");
var msRestNodeAuth = require("@azure/ms-rest-nodeauth");
// </snippet_imports>

/*
The application ID and secret of the service principal you are using to connect to the Azure Management Service.

To create a service principal with the Azure CLI, see:
/cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
To install the Azure CLI, see:
/cli/install-azure-cli?view=azure-cli-latest

To create a service principal with Azure PowerShell, see: 
https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
To install Azure PowerShell, see:
https://github.com/Azure/azure-powershell

When you create a service principal, you will see it has both an ID and an application ID.
For example, if you create a service principal with Azure PowerShell, it should look like the following:

Secret                : System.Security.SecureString
ServicePrincipalNames : {<application ID>, <application URL>}
ApplicationId         : <application ID>
ObjectType            : ServicePrincipal
DisplayName           : <name>
Id                    : <ID>
Type                  :

Be sure to use the service pricipal application ID, not simply the ID. 
*/

// <snippet_constants>
const service_principal_application_id = "TODO_REPLACE";
const service_principal_secret = "TODO_REPLACE";

/* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
const subscription_id = "TODO_REPLACE";

/* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
const tenant_id = "TODO_REPLACE";

/* The name of the Azure resource group in which you want to create the resource.
You can find resource groups in the Azure Dashboard under Home > Resource groups. */
const resource_group_name = "TODO_REPLACE";
// </snippet_constants>

// <snippet_list_avail>
async function list_available_kinds_skus_locations (client) {
    console.log ("Available SKUs:");
    var result = await client.list ();
    console.log("Kind\tSKU Name\tSKU Tier\tLocations");
    result.forEach (function (x) {
        var locations = x.locations.join(",");
        console.log(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations);
    });
}
// </snippet_list_avail>

// <snippet_list>
async function list_resources (client) {
    console.log ("Resources in resource group: " + resource_group_name);
    var result = await client.listByResourceGroup (resource_group_name);
    result.forEach (function (x) {
        console.log(x);
        console.log();
    });
}
// </snippet_list>

// <snippet_create>
function create_resource (client, resource_name, kind, sku_name, location) {
    console.log ("Creating resource: " + resource_name + "...");
    // The parameter "properties" must be an empty object.
    var parameters = { sku : { name: sku_name }, kind : kind, location : location, properties : {} };

    return client.create(resource_group_name, resource_name, parameters)
        .then((result) => {
        console.log("Resource created.");
        print();
        console.log("ID: " + result.id);
        console.log("Kind: " + result.kind);
        console.log();
        })
        .catch((err) =>{ 
                console.log(err)
        })
}
// </snippet_create>

// <snippet_delete>
async function delete_resource (client, resource_name) {
    console.log ("Deleting resource: " + resource_name + "...");
    await client.deleteMethod (resource_group_name, resource_name)
    console.log ("Resource deleted.");
    console.log ();
}
// </snippet_delete>

// <snippet_main_auth>
async function quickstart() {
    const credentials = await msRestNodeAuth.loginWithServicePrincipalSecret (service_principal_application_id, service_principal_secret, tenant_id);
    const client = new Arm.CognitiveServicesManagementClient (credentials, subscription_id);
    // Note Azure resources are also sometimes referred to as accounts.
    const accounts_client = new Arm.Accounts (client);
    const resource_skus_client = new Arm.ResourceSkus (client);
    // </snippet_main_auth>

    // <snippet_main_calls>
    // Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
    list_available_kinds_skus_locations (resource_skus_client);

    // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
    await create_resource (accounts_client, "test_resource", "TextTranslation", "F0", "Global");

    // Uncomment this to list all resources for your Azure account.
    list_resources (accounts_client);

    // Delete the resource.
    delete_resource (accounts_client, "test_resource");
}
// </snippet_main_calls>

// <snippet_main>
try {
    quickstart();
}
catch (error) {
    console.log(error);
}
// </snippet_main>
```

接下来，添加以下 `quickstart` 函数来处理程序的主要工作。 第一个代码块使用你在上面输入的凭据变量构造一个 CognitiveServicesManagementClient 对象。 所有 Azure 管理操作都需要此对象。

```javascript
// <snippet_imports>
"use strict";
/* To run this sample, install the following modules.
 * npm install @azure/arm-cognitiveservices
 * npm install @azure/ms-rest-js
 * npm install @azure/ms-rest-nodeauth
 */
var Arm = require("@azure/arm-cognitiveservices");
var msRestNodeAuth = require("@azure/ms-rest-nodeauth");
// </snippet_imports>

/*
The application ID and secret of the service principal you are using to connect to the Azure Management Service.

To create a service principal with the Azure CLI, see:
/cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
To install the Azure CLI, see:
/cli/install-azure-cli?view=azure-cli-latest

To create a service principal with Azure PowerShell, see: 
https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
To install Azure PowerShell, see:
https://github.com/Azure/azure-powershell

When you create a service principal, you will see it has both an ID and an application ID.
For example, if you create a service principal with Azure PowerShell, it should look like the following:

Secret                : System.Security.SecureString
ServicePrincipalNames : {<application ID>, <application URL>}
ApplicationId         : <application ID>
ObjectType            : ServicePrincipal
DisplayName           : <name>
Id                    : <ID>
Type                  :

Be sure to use the service pricipal application ID, not simply the ID. 
*/

// <snippet_constants>
const service_principal_application_id = "TODO_REPLACE";
const service_principal_secret = "TODO_REPLACE";

/* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
const subscription_id = "TODO_REPLACE";

/* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
const tenant_id = "TODO_REPLACE";

/* The name of the Azure resource group in which you want to create the resource.
You can find resource groups in the Azure Dashboard under Home > Resource groups. */
const resource_group_name = "TODO_REPLACE";
// </snippet_constants>

// <snippet_list_avail>
async function list_available_kinds_skus_locations (client) {
    console.log ("Available SKUs:");
    var result = await client.list ();
    console.log("Kind\tSKU Name\tSKU Tier\tLocations");
    result.forEach (function (x) {
        var locations = x.locations.join(",");
        console.log(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations);
    });
}
// </snippet_list_avail>

// <snippet_list>
async function list_resources (client) {
    console.log ("Resources in resource group: " + resource_group_name);
    var result = await client.listByResourceGroup (resource_group_name);
    result.forEach (function (x) {
        console.log(x);
        console.log();
    });
}
// </snippet_list>

// <snippet_create>
function create_resource (client, resource_name, kind, sku_name, location) {
    console.log ("Creating resource: " + resource_name + "...");
    // The parameter "properties" must be an empty object.
    var parameters = { sku : { name: sku_name }, kind : kind, location : location, properties : {} };

    return client.create(resource_group_name, resource_name, parameters)
        .then((result) => {
        console.log("Resource created.");
        print();
        console.log("ID: " + result.id);
        console.log("Kind: " + result.kind);
        console.log();
        })
        .catch((err) =>{ 
                console.log(err)
        })
}
// </snippet_create>

// <snippet_delete>
async function delete_resource (client, resource_name) {
    console.log ("Deleting resource: " + resource_name + "...");
    await client.deleteMethod (resource_group_name, resource_name)
    console.log ("Resource deleted.");
    console.log ();
}
// </snippet_delete>

// <snippet_main_auth>
async function quickstart() {
    const credentials = await msRestNodeAuth.loginWithServicePrincipalSecret (service_principal_application_id, service_principal_secret, tenant_id);
    const client = new Arm.CognitiveServicesManagementClient (credentials, subscription_id);
    // Note Azure resources are also sometimes referred to as accounts.
    const accounts_client = new Arm.Accounts (client);
    const resource_skus_client = new Arm.ResourceSkus (client);
    // </snippet_main_auth>

    // <snippet_main_calls>
    // Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
    list_available_kinds_skus_locations (resource_skus_client);

    // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
    await create_resource (accounts_client, "test_resource", "TextTranslation", "F0", "Global");

    // Uncomment this to list all resources for your Azure account.
    list_resources (accounts_client);

    // Delete the resource.
    delete_resource (accounts_client, "test_resource");
}
// </snippet_main_calls>

// <snippet_main>
try {
    quickstart();
}
catch (error) {
    console.log(error);
}
// </snippet_main>
```

## <a name="call-management-functions"></a>调用管理函数

将以下代码添加到 `quickstart` 函数的末尾，以列出可用资源、创建示例资源、列出拥有的资源，然后删除示例资源。 你将在后续步骤中定义这些函数。

## <a name="create-a-cognitive-services-resource"></a>创建认知服务资源

### <a name="choose-a-service-and-pricing-tier"></a>选择服务和定价层

创建新资源时，需要知道要使用的服务的种类，以及所需的[定价层](https://www.azure.cn/pricing/details/cognitive-services/)（或 SKU）。 创建资源时，将此信息和其他信息用作参数。 以下函数列出了可用的认知服务种类。

```javascript
// <snippet_imports>
"use strict";
/* To run this sample, install the following modules.
 * npm install @azure/arm-cognitiveservices
 * npm install @azure/ms-rest-js
 * npm install @azure/ms-rest-nodeauth
 */
var Arm = require("@azure/arm-cognitiveservices");
var msRestNodeAuth = require("@azure/ms-rest-nodeauth");
// </snippet_imports>

/*
The application ID and secret of the service principal you are using to connect to the Azure Management Service.

To create a service principal with the Azure CLI, see:
/cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
To install the Azure CLI, see:
/cli/install-azure-cli?view=azure-cli-latest

To create a service principal with Azure PowerShell, see: 
https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
To install Azure PowerShell, see:
https://github.com/Azure/azure-powershell

When you create a service principal, you will see it has both an ID and an application ID.
For example, if you create a service principal with Azure PowerShell, it should look like the following:

Secret                : System.Security.SecureString
ServicePrincipalNames : {<application ID>, <application URL>}
ApplicationId         : <application ID>
ObjectType            : ServicePrincipal
DisplayName           : <name>
Id                    : <ID>
Type                  :

Be sure to use the service pricipal application ID, not simply the ID. 
*/

// <snippet_constants>
const service_principal_application_id = "TODO_REPLACE";
const service_principal_secret = "TODO_REPLACE";

/* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
const subscription_id = "TODO_REPLACE";

/* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
const tenant_id = "TODO_REPLACE";

/* The name of the Azure resource group in which you want to create the resource.
You can find resource groups in the Azure Dashboard under Home > Resource groups. */
const resource_group_name = "TODO_REPLACE";
// </snippet_constants>

// <snippet_list_avail>
async function list_available_kinds_skus_locations (client) {
    console.log ("Available SKUs:");
    var result = await client.list ();
    console.log("Kind\tSKU Name\tSKU Tier\tLocations");
    result.forEach (function (x) {
        var locations = x.locations.join(",");
        console.log(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations);
    });
}
// </snippet_list_avail>

// <snippet_list>
async function list_resources (client) {
    console.log ("Resources in resource group: " + resource_group_name);
    var result = await client.listByResourceGroup (resource_group_name);
    result.forEach (function (x) {
        console.log(x);
        console.log();
    });
}
// </snippet_list>

// <snippet_create>
function create_resource (client, resource_name, kind, sku_name, location) {
    console.log ("Creating resource: " + resource_name + "...");
    // The parameter "properties" must be an empty object.
    var parameters = { sku : { name: sku_name }, kind : kind, location : location, properties : {} };

    return client.create(resource_group_name, resource_name, parameters)
        .then((result) => {
        console.log("Resource created.");
        print();
        console.log("ID: " + result.id);
        console.log("Kind: " + result.kind);
        console.log();
        })
        .catch((err) =>{ 
                console.log(err)
        })
}
// </snippet_create>

// <snippet_delete>
async function delete_resource (client, resource_name) {
    console.log ("Deleting resource: " + resource_name + "...");
    await client.deleteMethod (resource_group_name, resource_name)
    console.log ("Resource deleted.");
    console.log ();
}
// </snippet_delete>

// <snippet_main_auth>
async function quickstart() {
    const credentials = await msRestNodeAuth.loginWithServicePrincipalSecret (service_principal_application_id, service_principal_secret, tenant_id);
    const client = new Arm.CognitiveServicesManagementClient (credentials, subscription_id);
    // Note Azure resources are also sometimes referred to as accounts.
    const accounts_client = new Arm.Accounts (client);
    const resource_skus_client = new Arm.ResourceSkus (client);
    // </snippet_main_auth>

    // <snippet_main_calls>
    // Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
    list_available_kinds_skus_locations (resource_skus_client);

    // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
    await create_resource (accounts_client, "test_resource", "TextTranslation", "F0", "Global");

    // Uncomment this to list all resources for your Azure account.
    list_resources (accounts_client);

    // Delete the resource.
    delete_resource (accounts_client, "test_resource");
}
// </snippet_main_calls>

// <snippet_main>
try {
    quickstart();
}
catch (error) {
    console.log(error);
}
// </snippet_main>
```

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="create-a-cognitive-services-resource"></a>创建认知服务资源

若要创建并订阅新的认知服务资源，请使用 Create 函数。 此函数向传入的资源组添加新的可计费资源。 创建新资源时，需要知道要使用的服务的种类，以及其定价层（或 SKU）和 Azure 位置。 下面的函数使用所有这些参数并创建资源。

```javascript
// <snippet_imports>
"use strict";
/* To run this sample, install the following modules.
 * npm install @azure/arm-cognitiveservices
 * npm install @azure/ms-rest-js
 * npm install @azure/ms-rest-nodeauth
 */
var Arm = require("@azure/arm-cognitiveservices");
var msRestNodeAuth = require("@azure/ms-rest-nodeauth");
// </snippet_imports>

/*
The application ID and secret of the service principal you are using to connect to the Azure Management Service.

To create a service principal with the Azure CLI, see:
/cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
To install the Azure CLI, see:
/cli/install-azure-cli?view=azure-cli-latest

To create a service principal with Azure PowerShell, see: 
https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
To install Azure PowerShell, see:
https://github.com/Azure/azure-powershell

When you create a service principal, you will see it has both an ID and an application ID.
For example, if you create a service principal with Azure PowerShell, it should look like the following:

Secret                : System.Security.SecureString
ServicePrincipalNames : {<application ID>, <application URL>}
ApplicationId         : <application ID>
ObjectType            : ServicePrincipal
DisplayName           : <name>
Id                    : <ID>
Type                  :

Be sure to use the service pricipal application ID, not simply the ID. 
*/

// <snippet_constants>
const service_principal_application_id = "TODO_REPLACE";
const service_principal_secret = "TODO_REPLACE";

/* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
const subscription_id = "TODO_REPLACE";

/* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
const tenant_id = "TODO_REPLACE";

/* The name of the Azure resource group in which you want to create the resource.
You can find resource groups in the Azure Dashboard under Home > Resource groups. */
const resource_group_name = "TODO_REPLACE";
// </snippet_constants>

// <snippet_list_avail>
async function list_available_kinds_skus_locations (client) {
    console.log ("Available SKUs:");
    var result = await client.list ();
    console.log("Kind\tSKU Name\tSKU Tier\tLocations");
    result.forEach (function (x) {
        var locations = x.locations.join(",");
        console.log(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations);
    });
}
// </snippet_list_avail>

// <snippet_list>
async function list_resources (client) {
    console.log ("Resources in resource group: " + resource_group_name);
    var result = await client.listByResourceGroup (resource_group_name);
    result.forEach (function (x) {
        console.log(x);
        console.log();
    });
}
// </snippet_list>

// <snippet_create>
function create_resource (client, resource_name, kind, sku_name, location) {
    console.log ("Creating resource: " + resource_name + "...");
    // The parameter "properties" must be an empty object.
    var parameters = { sku : { name: sku_name }, kind : kind, location : location, properties : {} };

    return client.create(resource_group_name, resource_name, parameters)
        .then((result) => {
        console.log("Resource created.");
        print();
        console.log("ID: " + result.id);
        console.log("Kind: " + result.kind);
        console.log();
        })
        .catch((err) =>{ 
                console.log(err)
        })
}
// </snippet_create>

// <snippet_delete>
async function delete_resource (client, resource_name) {
    console.log ("Deleting resource: " + resource_name + "...");
    await client.deleteMethod (resource_group_name, resource_name)
    console.log ("Resource deleted.");
    console.log ();
}
// </snippet_delete>

// <snippet_main_auth>
async function quickstart() {
    const credentials = await msRestNodeAuth.loginWithServicePrincipalSecret (service_principal_application_id, service_principal_secret, tenant_id);
    const client = new Arm.CognitiveServicesManagementClient (credentials, subscription_id);
    // Note Azure resources are also sometimes referred to as accounts.
    const accounts_client = new Arm.Accounts (client);
    const resource_skus_client = new Arm.ResourceSkus (client);
    // </snippet_main_auth>

    // <snippet_main_calls>
    // Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
    list_available_kinds_skus_locations (resource_skus_client);

    // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
    await create_resource (accounts_client, "test_resource", "TextTranslation", "F0", "Global");

    // Uncomment this to list all resources for your Azure account.
    list_resources (accounts_client);

    // Delete the resource.
    delete_resource (accounts_client, "test_resource");
}
// </snippet_main_calls>

// <snippet_main>
try {
    quickstart();
}
catch (error) {
    console.log(error);
}
// </snippet_main>
```

## <a name="view-your-resources"></a>查看资源

若要查看 Azure 帐户下的所有资源（跨所有资源组），请使用以下函数：

```javascript
// <snippet_imports>
"use strict";
/* To run this sample, install the following modules.
 * npm install @azure/arm-cognitiveservices
 * npm install @azure/ms-rest-js
 * npm install @azure/ms-rest-nodeauth
 */
var Arm = require("@azure/arm-cognitiveservices");
var msRestNodeAuth = require("@azure/ms-rest-nodeauth");
// </snippet_imports>

/*
The application ID and secret of the service principal you are using to connect to the Azure Management Service.

To create a service principal with the Azure CLI, see:
/cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
To install the Azure CLI, see:
/cli/install-azure-cli?view=azure-cli-latest

To create a service principal with Azure PowerShell, see: 
https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
To install Azure PowerShell, see:
https://github.com/Azure/azure-powershell

When you create a service principal, you will see it has both an ID and an application ID.
For example, if you create a service principal with Azure PowerShell, it should look like the following:

Secret                : System.Security.SecureString
ServicePrincipalNames : {<application ID>, <application URL>}
ApplicationId         : <application ID>
ObjectType            : ServicePrincipal
DisplayName           : <name>
Id                    : <ID>
Type                  :

Be sure to use the service pricipal application ID, not simply the ID. 
*/

// <snippet_constants>
const service_principal_application_id = "TODO_REPLACE";
const service_principal_secret = "TODO_REPLACE";

/* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
const subscription_id = "TODO_REPLACE";

/* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
const tenant_id = "TODO_REPLACE";

/* The name of the Azure resource group in which you want to create the resource.
You can find resource groups in the Azure Dashboard under Home > Resource groups. */
const resource_group_name = "TODO_REPLACE";
// </snippet_constants>

// <snippet_list_avail>
async function list_available_kinds_skus_locations (client) {
    console.log ("Available SKUs:");
    var result = await client.list ();
    console.log("Kind\tSKU Name\tSKU Tier\tLocations");
    result.forEach (function (x) {
        var locations = x.locations.join(",");
        console.log(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations);
    });
}
// </snippet_list_avail>

// <snippet_list>
async function list_resources (client) {
    console.log ("Resources in resource group: " + resource_group_name);
    var result = await client.listByResourceGroup (resource_group_name);
    result.forEach (function (x) {
        console.log(x);
        console.log();
    });
}
// </snippet_list>

// <snippet_create>
function create_resource (client, resource_name, kind, sku_name, location) {
    console.log ("Creating resource: " + resource_name + "...");
    // The parameter "properties" must be an empty object.
    var parameters = { sku : { name: sku_name }, kind : kind, location : location, properties : {} };

    return client.create(resource_group_name, resource_name, parameters)
        .then((result) => {
        console.log("Resource created.");
        print();
        console.log("ID: " + result.id);
        console.log("Kind: " + result.kind);
        console.log();
        })
        .catch((err) =>{ 
                console.log(err)
        })
}
// </snippet_create>

// <snippet_delete>
async function delete_resource (client, resource_name) {
    console.log ("Deleting resource: " + resource_name + "...");
    await client.deleteMethod (resource_group_name, resource_name)
    console.log ("Resource deleted.");
    console.log ();
}
// </snippet_delete>

// <snippet_main_auth>
async function quickstart() {
    const credentials = await msRestNodeAuth.loginWithServicePrincipalSecret (service_principal_application_id, service_principal_secret, tenant_id);
    const client = new Arm.CognitiveServicesManagementClient (credentials, subscription_id);
    // Note Azure resources are also sometimes referred to as accounts.
    const accounts_client = new Arm.Accounts (client);
    const resource_skus_client = new Arm.ResourceSkus (client);
    // </snippet_main_auth>

    // <snippet_main_calls>
    // Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
    list_available_kinds_skus_locations (resource_skus_client);

    // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
    await create_resource (accounts_client, "test_resource", "TextTranslation", "F0", "Global");

    // Uncomment this to list all resources for your Azure account.
    list_resources (accounts_client);

    // Delete the resource.
    delete_resource (accounts_client, "test_resource");
}
// </snippet_main_calls>

// <snippet_main>
try {
    quickstart();
}
catch (error) {
    console.log(error);
}
// </snippet_main>
```

## <a name="delete-a-resource"></a>删除资源

下面的函数从给定的资源组中删除指定的资源。

```javascript
// <snippet_imports>
"use strict";
/* To run this sample, install the following modules.
 * npm install @azure/arm-cognitiveservices
 * npm install @azure/ms-rest-js
 * npm install @azure/ms-rest-nodeauth
 */
var Arm = require("@azure/arm-cognitiveservices");
var msRestNodeAuth = require("@azure/ms-rest-nodeauth");
// </snippet_imports>

/*
The application ID and secret of the service principal you are using to connect to the Azure Management Service.

To create a service principal with the Azure CLI, see:
/cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
To install the Azure CLI, see:
/cli/install-azure-cli?view=azure-cli-latest

To create a service principal with Azure PowerShell, see: 
https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
To install Azure PowerShell, see:
https://github.com/Azure/azure-powershell

When you create a service principal, you will see it has both an ID and an application ID.
For example, if you create a service principal with Azure PowerShell, it should look like the following:

Secret                : System.Security.SecureString
ServicePrincipalNames : {<application ID>, <application URL>}
ApplicationId         : <application ID>
ObjectType            : ServicePrincipal
DisplayName           : <name>
Id                    : <ID>
Type                  :

Be sure to use the service pricipal application ID, not simply the ID. 
*/

// <snippet_constants>
const service_principal_application_id = "TODO_REPLACE";
const service_principal_secret = "TODO_REPLACE";

/* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
const subscription_id = "TODO_REPLACE";

/* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
const tenant_id = "TODO_REPLACE";

/* The name of the Azure resource group in which you want to create the resource.
You can find resource groups in the Azure Dashboard under Home > Resource groups. */
const resource_group_name = "TODO_REPLACE";
// </snippet_constants>

// <snippet_list_avail>
async function list_available_kinds_skus_locations (client) {
    console.log ("Available SKUs:");
    var result = await client.list ();
    console.log("Kind\tSKU Name\tSKU Tier\tLocations");
    result.forEach (function (x) {
        var locations = x.locations.join(",");
        console.log(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations);
    });
}
// </snippet_list_avail>

// <snippet_list>
async function list_resources (client) {
    console.log ("Resources in resource group: " + resource_group_name);
    var result = await client.listByResourceGroup (resource_group_name);
    result.forEach (function (x) {
        console.log(x);
        console.log();
    });
}
// </snippet_list>

// <snippet_create>
function create_resource (client, resource_name, kind, sku_name, location) {
    console.log ("Creating resource: " + resource_name + "...");
    // The parameter "properties" must be an empty object.
    var parameters = { sku : { name: sku_name }, kind : kind, location : location, properties : {} };

    return client.create(resource_group_name, resource_name, parameters)
        .then((result) => {
        console.log("Resource created.");
        print();
        console.log("ID: " + result.id);
        console.log("Kind: " + result.kind);
        console.log();
        })
        .catch((err) =>{ 
                console.log(err)
        })
}
// </snippet_create>

// <snippet_delete>
async function delete_resource (client, resource_name) {
    console.log ("Deleting resource: " + resource_name + "...");
    await client.deleteMethod (resource_group_name, resource_name)
    console.log ("Resource deleted.");
    console.log ();
}
// </snippet_delete>

// <snippet_main_auth>
async function quickstart() {
    const credentials = await msRestNodeAuth.loginWithServicePrincipalSecret (service_principal_application_id, service_principal_secret, tenant_id);
    const client = new Arm.CognitiveServicesManagementClient (credentials, subscription_id);
    // Note Azure resources are also sometimes referred to as accounts.
    const accounts_client = new Arm.Accounts (client);
    const resource_skus_client = new Arm.ResourceSkus (client);
    // </snippet_main_auth>

    // <snippet_main_calls>
    // Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
    list_available_kinds_skus_locations (resource_skus_client);

    // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
    await create_resource (accounts_client, "test_resource", "TextTranslation", "F0", "Global");

    // Uncomment this to list all resources for your Azure account.
    list_resources (accounts_client);

    // Delete the resource.
    delete_resource (accounts_client, "test_resource");
}
// </snippet_main_calls>

// <snippet_main>
try {
    quickstart();
}
catch (error) {
    console.log(error);
}
// </snippet_main>
```

## <a name="run-the-application"></a>运行应用程序

将以下代码添加到脚本底部，以调用包含错误处理的主 `quickstart` 函数。

```javascript
// <snippet_imports>
"use strict";
/* To run this sample, install the following modules.
 * npm install @azure/arm-cognitiveservices
 * npm install @azure/ms-rest-js
 * npm install @azure/ms-rest-nodeauth
 */
var Arm = require("@azure/arm-cognitiveservices");
var msRestNodeAuth = require("@azure/ms-rest-nodeauth");
// </snippet_imports>

/*
The application ID and secret of the service principal you are using to connect to the Azure Management Service.

To create a service principal with the Azure CLI, see:
/cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
To install the Azure CLI, see:
/cli/install-azure-cli?view=azure-cli-latest

To create a service principal with Azure PowerShell, see: 
https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
To install Azure PowerShell, see:
https://github.com/Azure/azure-powershell

When you create a service principal, you will see it has both an ID and an application ID.
For example, if you create a service principal with Azure PowerShell, it should look like the following:

Secret                : System.Security.SecureString
ServicePrincipalNames : {<application ID>, <application URL>}
ApplicationId         : <application ID>
ObjectType            : ServicePrincipal
DisplayName           : <name>
Id                    : <ID>
Type                  :

Be sure to use the service pricipal application ID, not simply the ID. 
*/

// <snippet_constants>
const service_principal_application_id = "TODO_REPLACE";
const service_principal_secret = "TODO_REPLACE";

/* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
const subscription_id = "TODO_REPLACE";

/* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
const tenant_id = "TODO_REPLACE";

/* The name of the Azure resource group in which you want to create the resource.
You can find resource groups in the Azure Dashboard under Home > Resource groups. */
const resource_group_name = "TODO_REPLACE";
// </snippet_constants>

// <snippet_list_avail>
async function list_available_kinds_skus_locations (client) {
    console.log ("Available SKUs:");
    var result = await client.list ();
    console.log("Kind\tSKU Name\tSKU Tier\tLocations");
    result.forEach (function (x) {
        var locations = x.locations.join(",");
        console.log(x.kind + "\t" + x.name + "\t" + x.tier + "\t" + locations);
    });
}
// </snippet_list_avail>

// <snippet_list>
async function list_resources (client) {
    console.log ("Resources in resource group: " + resource_group_name);
    var result = await client.listByResourceGroup (resource_group_name);
    result.forEach (function (x) {
        console.log(x);
        console.log();
    });
}
// </snippet_list>

// <snippet_create>
function create_resource (client, resource_name, kind, sku_name, location) {
    console.log ("Creating resource: " + resource_name + "...");
    // The parameter "properties" must be an empty object.
    var parameters = { sku : { name: sku_name }, kind : kind, location : location, properties : {} };

    return client.create(resource_group_name, resource_name, parameters)
        .then((result) => {
        console.log("Resource created.");
        print();
        console.log("ID: " + result.id);
        console.log("Kind: " + result.kind);
        console.log();
        })
        .catch((err) =>{ 
                console.log(err)
        })
}
// </snippet_create>

// <snippet_delete>
async function delete_resource (client, resource_name) {
    console.log ("Deleting resource: " + resource_name + "...");
    await client.deleteMethod (resource_group_name, resource_name)
    console.log ("Resource deleted.");
    console.log ();
}
// </snippet_delete>

// <snippet_main_auth>
async function quickstart() {
    const credentials = await msRestNodeAuth.loginWithServicePrincipalSecret (service_principal_application_id, service_principal_secret, tenant_id);
    const client = new Arm.CognitiveServicesManagementClient (credentials, subscription_id);
    // Note Azure resources are also sometimes referred to as accounts.
    const accounts_client = new Arm.Accounts (client);
    const resource_skus_client = new Arm.ResourceSkus (client);
    // </snippet_main_auth>

    // <snippet_main_calls>
    // Uncomment this to list all available resource kinds, SKUs, and locations for your Azure account.
    list_available_kinds_skus_locations (resource_skus_client);

    // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
    await create_resource (accounts_client, "test_resource", "TextTranslation", "F0", "Global");

    // Uncomment this to list all resources for your Azure account.
    list_resources (accounts_client);

    // Delete the resource.
    delete_resource (accounts_client, "test_resource");
}
// </snippet_main_calls>

// <snippet_main>
try {
    quickstart();
}
catch (error) {
    console.log(error);
}
// </snippet_main>
```

然后，在控制台窗口中，使用 `node` 命令运行应用程序。

```console
node index.js
```

## <a name="see-also"></a>另请参阅

* [Azure 管理 SDK 参考文档](https://docs.microsoft.com/javascript/api/@azure/arm-cognitiveservices/?view=azure-node-latest)
* [什么是 Azure 认知服务？](../../Welcome.md)
* [对 Azure 认知服务的请求进行身份验证](../../authentication.md)
* [使用 Azure 门户创建新资源](../../cognitive-services-apis-create-account.md)

