---
title: 模板函数
description: 介绍在 Azure Resource Manager 模板中检索值、处理字符串和数字以及检索部署信息时所用的函数。
ms.topic: conceptual
origin.date: 10/12/2020
author: rockboyfor
ms.date: 10/26/2020
ms.author: v-yeche
ms.openlocfilehash: bbf13c80f295b9eb57d6c9241afd53ed416c2722
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470401"
---
# <a name="arm-template-functions"></a>ARM 模板函数

本文介绍可以在 Azure 资源管理器 (ARM) 模板中使用的所有函数。 若要了解如何在模板中使用函数，请参阅[模板语法](template-expressions.md)。

若要创建自己的函数，请参阅[用户定义函数](template-syntax.md#functions)。

大多数函数在部署到资源组、订阅、管理组或租户时工作方式相同。 某些函数并非可以在所有范围内使用。 下表对这些函数进行了说明。

<a name="array" aria-hidden="true"></a>
<a name="concatarray" aria-hidden="true"></a>
<a name="contains" aria-hidden="true"></a>
<a name="createarray" aria-hidden="true"></a>
<a name="empty" aria-hidden="true"></a>
<a name="first" aria-hidden="true"></a>
<a name="intersection" aria-hidden="true"></a>
<a name="last" aria-hidden="true"></a>
<a name="length" aria-hidden="true"></a>
<a name="min" aria-hidden="true"></a>
<a name="max" aria-hidden="true"></a>
<a name="range" aria-hidden="true"></a>
<a name="skip" aria-hidden="true"></a>
<a name="take" aria-hidden="true"></a>
<a name="union" aria-hidden="true"></a>

## <a name="array-functions"></a>数组函数

Resource Manager 提供以下用于处理数组的函数。

* [array](template-functions-array.md#array)
* [concat](template-functions-array.md#concat)
* [contains](template-functions-array.md#contains)
* [createArray](template-functions-array.md#createarray)
* [empty](template-functions-array.md#empty)
* [first](template-functions-array.md#first)
* [intersection](template-functions-array.md#intersection)
* [last](template-functions-array.md#last)
* [length](template-functions-array.md#length)
* [min](template-functions-array.md#min)
* [max](template-functions-array.md#max)
* [range](template-functions-array.md#range)
* [skip](template-functions-array.md#skip)
* [take](template-functions-array.md#take)
* [union](template-functions-array.md#union)

<a name="coalesce" aria-hidden="true"></a>
<a name="equals" aria-hidden="true"></a>
<a name="less" aria-hidden="true"></a>
<a name="lessorequals" aria-hidden="true"></a>
<a name="greater" aria-hidden="true"></a>
<a name="greaterorequals" aria-hidden="true"></a>

## <a name="comparison-functions"></a>比较函数

Resource Manager 提供了多个用于在模板中进行比较的函数。

* [coalesce](template-functions-comparison.md#coalesce)
* [equals](template-functions-comparison.md#equals)
* [less](template-functions-comparison.md#less)
* [lessOrEquals](template-functions-comparison.md#lessorequals)
* [greater](template-functions-comparison.md#greater)
* [greaterOrEquals](template-functions-comparison.md#greaterorequals)

<a name="deployment" aria-hidden="true"></a>
<a name="parameters" aria-hidden="true"></a>
<a name="variables" aria-hidden="true"></a>

## <a name="date-functions"></a>日期函数

资源管理器提供了以下用于处理日期的函数。

* [dateTimeAdd](template-functions-date.md#datetimeadd)
* [utcNow](template-functions-date.md#utcnow)

## <a name="deployment-value-functions"></a>部署值函数

Resource Manager 提供以下函数，用于从与部署相关的模板和值部分获取值：

* [部署](template-functions-deployment.md#deployment)
* [环境](template-functions-deployment.md#environment)
* [parameters](template-functions-deployment.md#parameters)
* [variables](template-functions-deployment.md#variables)

<a name="and" aria-hidden="true"></a>
<a name="bool" aria-hidden="true"></a>
<a name="if" aria-hidden="true"></a>
<a name="not" aria-hidden="true"></a>
<a name="or" aria-hidden="true"></a>

## <a name="logical-functions"></a>逻辑函数

资源管理器提供以下用于处理逻辑条件的函数：

* [and](template-functions-logical.md#and)
* [bool](template-functions-logical.md#bool)
* [false](template-functions-logical.md#false)
* [if](template-functions-logical.md#if)
* [not](template-functions-logical.md#not)
* [or](template-functions-logical.md#or)
* true

<a name="add" aria-hidden="true"></a>
<a name="copyindex" aria-hidden="true"></a>
<a name="div" aria-hidden="true"></a>
<a name="float" aria-hidden="true"></a>
<a name="int" aria-hidden="true"></a>
<a name="minint" aria-hidden="true"></a>
<a name="maxint" aria-hidden="true"></a>
<a name="mod" aria-hidden="true"></a>
<a name="mul" aria-hidden="true"></a>
<a name="sub" aria-hidden="true"></a>

## <a name="numeric-functions"></a>数值函数

Resource Manager 提供以下用于处理整数的函数：

* [add](template-functions-numeric.md#add)
* [copyIndex](template-functions-numeric.md#copyindex)
* [div](template-functions-numeric.md#div)
* [float](template-functions-numeric.md#float)
* [int](template-functions-numeric.md#int)
* [min](template-functions-numeric.md#min)
* [max](template-functions-numeric.md#max)
* [mod](template-functions-numeric.md#mod)
* [mul](template-functions-numeric.md#mul)
* [sub](template-functions-numeric.md#sub)

<a name="json" aria-hidden="true"></a>

## <a name="object-functions"></a>对象函数

Resource Manager 提供以下用于处理对象的函数。

* [contains](template-functions-object.md#contains)
* [createObject](template-functions-object.md#createobject)
* [empty](template-functions-object.md#empty)
* [intersection](template-functions-object.md#intersection)
* [json](template-functions-object.md#json)
* [length](template-functions-object.md#length)
* [null](template-functions-object.md#null)
* [union](template-functions-object.md#union)

<a name="extensionResourceId" aria-hidden="true"></a>
<a name="listkeys" aria-hidden="true"></a>
<a name="list" aria-hidden="true"></a>
<a name="providers" aria-hidden="true"></a>
<a name="reference" aria-hidden="true"></a>
<a name="resourcegroup" aria-hidden="true"></a>
<a name="resourceid" aria-hidden="true"></a>
<a name="subscription" aria-hidden="true"></a>
<a name="subscriptionResourceId" aria-hidden="true"></a>
<a name="tenantResourceId" aria-hidden="true"></a>

## <a name="resource-functions"></a>Resource functions

Resource Manager 提供以下用于获取资源值的函数：

* [extensionResourceId](template-functions-resource.md#extensionresourceid)
* [listAccountSas](template-functions-resource.md#list)
* [listKeys](template-functions-resource.md#listkeys)
* [listSecrets](template-functions-resource.md#list)
* [list*](template-functions-resource.md#list)
    
    <!--Not Available on * [pickZones](template-functions-resource.md#pickzones)-->
    
* [providers](template-functions-resource.md#providers)
* [reference](template-functions-resource.md#reference)
* [resourceGroup](template-functions-resource.md#resourcegroup) - 只能在部署到资源组时使用。
* [resourceId](template-functions-resource.md#resourceid) - 可以在任何范围内使用，但有效参数会根据范围而发生变化。
* [subscription](template-functions-resource.md#subscription) - 只能在部署到资源组或订阅时使用。
* [subscriptionResourceId](template-functions-resource.md#subscriptionresourceid)
* [tenantResourceId](template-functions-resource.md#tenantresourceid)

<a name="base64" aria-hidden="true"></a>
<a name="base64tojson" aria-hidden="true"></a>
<a name="base64tostring" aria-hidden="true"></a>
<a name="concat" aria-hidden="true"></a>
<a name="containsstring" aria-hidden="true"></a>
<a name="datauri" aria-hidden="true"></a>
<a name="datauritostring" aria-hidden="true"></a>
<a name="emptystring" aria-hidden="true"></a>
<a name="endswith" aria-hidden="true"></a>
<a name="firststring" aria-hidden="true"></a>
<a name="guid" aria-hidden="true"></a>
<a name="indexof" aria-hidden="true"></a>
<a name="laststring" aria-hidden="true"></a>
<a name="lastindexof" aria-hidden="true"></a>
<a name="lengthstring" aria-hidden="true"></a>
<a name="padleft" aria-hidden="true"></a>
<a name="replace" aria-hidden="true"></a>
<a name="skipstring" aria-hidden="true"></a>
<a name="split" aria-hidden="true"></a>
<a name="startswith" aria-hidden="true"></a>
<a name="string" aria-hidden="true"></a>
<a name="substring" aria-hidden="true"></a>
<a name="takestring" aria-hidden="true"></a>
<a name="tolower" aria-hidden="true"></a>
<a name="toupper" aria-hidden="true"></a>
<a name="trim" aria-hidden="true"></a>
<a name="uniquestring" aria-hidden="true"></a>
<a name="uri" aria-hidden="true"></a>
<a name="uricomponent" aria-hidden="true"></a>
<a name="uricomponenttostring" aria-hidden="true"></a>

## <a name="string-functions"></a>字符串函数

Resource Manager 提供以下用于处理字符串的函数：

* [base64](template-functions-string.md#base64)
* [base64ToJson](template-functions-string.md#base64tojson)
* [base64ToString](template-functions-string.md#base64tostring)
* [concat](template-functions-string.md#concat)
* [contains](template-functions-string.md#contains)
* [dataUri](template-functions-string.md#datauri)
* [dataUriToString](template-functions-string.md#datauritostring)
* [empty](template-functions-string.md#empty)
* [endsWith](template-functions-string.md#endswith)
* [first](template-functions-string.md#first)
* [format](template-functions-string.md#format)
* [guid](template-functions-string.md#guid)
* [indexOf](template-functions-string.md#indexof)
* [last](template-functions-string.md#last)
* [lastIndexOf](template-functions-string.md#lastindexof)
* [length](template-functions-string.md#length)
* [newGuid](template-functions-string.md#newguid)
* [padLeft](template-functions-string.md#padleft)
* [replace](template-functions-string.md#replace)
* [skip](template-functions-string.md#skip)
* [split](template-functions-string.md#split)
* [startsWith](template-functions-string.md#startswith)
* [string](template-functions-string.md#string)
* [substring](template-functions-string.md#substring)
* [take](template-functions-string.md#take)
* [toLower](template-functions-string.md#tolower)
* [toUpper](template-functions-string.md#toupper)
* [trim](template-functions-string.md#trim)
* [uniqueString](template-functions-string.md#uniquestring)
* [uri](template-functions-string.md#uri)
* [uriComponent](template-functions-string.md#uricomponent)
* [uriComponentToString](template-functions-string.md#uricomponenttostring)

## <a name="next-steps"></a>后续步骤

* 有关 ARM 模板中各节的说明，请参阅[创作 ARM 模板](template-syntax.md)
* 若要合并多个模板，请参阅 [Using linked templates with Azure Resource Manager](linked-templates.md)（将链接模板与 Azure 资源管理器配合使用）
* 若要在创建资源类型时迭代指定的次数，请参阅[在 Azure 资源管理器中创建多个资源实例](copy-resources.md)。
* 若要了解如何部署已创建的模板，请参阅[使用 ARM 模板部署应用程序](deploy-powershell.md)

<!-- Update_Description: update meta properties, wording update, update link -->