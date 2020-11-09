---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/24/2020
title: 管理群集策略 - Azure Databricks
description: 了解如何根据一组预定义的规则，使用策略来限制用户和用户组的群集创建功能。
ms.openlocfilehash: 53643d5d77ccf645bbdf613870340d25720b4bf9
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106510"
---
# <a name="manage-cluster-policies"></a>管理群集策略

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../release-notes/release-types.md)提供。

> [!IMPORTANT]
>
> ## <a name="bias-free-communication"></a>无偏差通信
>
> Microsoft 支持多样化的包容性环境。 本文包含对单词 blacklist 的引用。 Microsoft 的[无偏差通信风格指南](https://docs.microsoft.com/style-guide/bias-free-communication)将其视为排他性单词。 本文使用该单词旨在保持一致性，因为目前软件中使用的是该单词。 如果软件更新后删除了该单词，则本文也将更新以保持一致。

[群集策略](../../clusters/configure.md)基于一组规则限制配置群集的能力。
策略规则会限制可用于创建群集的属性或属性值。
群集策略具有将策略的使用限制到特定用户和组的 ACL。

通过群集策略可以实现以下目的：

* 限制用户使用指定的设置创建群集。
* 简化用户界面，支持更多的用户创建自己的群集（方法是修正和隐藏某些值）。
* 通过限制每个群集的最大成本来控制成本（方法是对其值会影响每小时价格的属性设置限制）。

群集策略权限在用户创建群集时会限制用户可以在“策略”下拉列表中选择的策略：

* 具有[群集创建权限](../access-control/cluster-acl.md#cluster-create-permission)的用户可以选择[自由形式](../../clusters/configure.md#cluster-policy)的策略，并创建可以充分配置的群集。
* 具有群集创建权限和群集策略访问权限的用户可以选择“自由形式”的策略和他们有权访问的策略。
* 只有群集策略访问权限的用户可以选择他们有权访问的策略。

> [!NOTE]
>
> 如果尚未[在工作区中创建](#create-a-cluster-policy)任何策略，则不会显示“策略”下拉列表。

只有管理员用户才能创建、编辑和删除策略。 管理员用户也有权访问所有策略。

本文重点介绍如何使用 UI 管理策略。 你还可以使用[群集策略 API](../../dev-tools/api/latest/policies.md) 来管理策略。

## <a name="requirements"></a>要求

群集策略需要 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)。

## <a name="enforcement-rules"></a>强制规则

可以在策略规则中表达以下类型的约束：

* 固定值且已禁用控制元素
* 固定值且控件隐藏在 UI 中（值在 JSON 视图中可见）
* 属性值限制为一组值（允许列表或阻止列表）
* 属性值与给定正则表达式匹配
* 数值属性限定在特定范围内
* 默认值启用控件的 UI 使用

## <a name="managed-cluster-attributes"></a>托管群集属性

群集策略支持使用[群集 API](../../dev-tools/api/latest/clusters.md) 控制的所有[群集属性](#cluster-policy-attribute-paths)。
每个字段支持的特定限制类型可能各有不同（取决于其类型和与群集窗体 UI 元素的关系）。

此外，群集策略还支持以下[综合属性](#cluster-policy-virtual-attribute-paths)：

* “最大 DBU 小时”指标，它是群集每小时可以使用的最大 DBU。 此指标是在单个群集级别控制成本的直接方法。
* 对创建群集的源的限制：作业服务（作业群集）、群集 UI、群集 REST API（通用群集）。

## <a name="unmanaged-cluster-attributes"></a>非托管群集属性

无法在群集策略中限制以下群集属性：

* 由[库 API](../../dev-tools/api/latest/libraries.md) 处理的库。 解决方法是使用[自定义容器](../../clusters/custom-containers.md#containers)或[初始化脚本](../../clusters/init-scripts.md)。
* 每个用户（总共或同时）创建的群集数。 策略的范围是单个群集，因此不知道用户创建的群集数。
* 由单独的 API 处理的群集权限 (ACL)。

## <a name="define-a-cluster-policy"></a>定义群集策略

可以在创建群集策略时添加的 [JSON 策略定义](#policy-def)中定义群集策略。

## <a name="create-a-cluster-policy"></a>创建群集策略

可以使用群集策略 UI 或[群集策略 API](../../dev-tools/api/latest/policies.md) 创建群集策略。 若要使用 UI 创建群集策略，请执行以下操作：

1. 单击“群集”图标 ![“群集”图标](../../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击“群集策略”选项卡。

   > [!div class="mx-imgBorder"]
   > ![创建策略](../../_static/images/admin-cluster-management/cluster-policy-create.png)

3. 单击“创建策略”按钮。

   > [!div class="mx-imgBorder"]
   > ![创建策略](../../_static/images/admin-cluster-management/cluster-policy.png)

4. 为策略命名。 策略名称不区分大小写。
5. 在“定义”选项卡中，粘贴[策略定义](#policy-def)。
6. 单击 **创建** 。

## <a name="manage-cluster-policy-permissions"></a>管理群集策略权限

根据定义，管理员拥有所有策略的权限。 可以使用群集策略 UI 或[群集策略权限 API](../../dev-tools/api/latest/policies.md#cluster-policy-permissions-api) 管理群集策略权限。

### <a name="add-a-cluster-policy-permission"></a>添加群集策略权限

若要使用 UI 添加群集策略权限，请执行以下操作：

1. 单击“群集”图标 ![“群集”图标](../../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击“群集策略”选项卡。
3. 单击“权限”选项卡。
4. 在“名称”列中，选择一个主体。

   > [!div class="mx-imgBorder"]
   > ![策略权限主体](../../_static/images/admin-cluster-management/policy-permission-principal.png)

5. 在“权限”列中，选择[权限](../../dev-tools/api/latest/policies.md#permissionlevel)：

   > [!div class="mx-imgBorder"]
   > ![策略权限](../../_static/images/admin-cluster-management/policy-permission.png)

6. 单击 **添加** 。

### <a name="delete-a-cluster-policy-permission"></a>删除群集策略权限

若要使用 UI 删除群集策略权限，请执行以下操作：

1. 单击“群集”图标 ![“群集”图标](../../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击“群集策略”选项卡。
3. 单击“权限”选项卡。
4. 单击 ![“删除”图标](../../_static/images/clusters/delete-icon.png) 权限行中的图标。

## <a name="edit-a-cluster-policy-using-the-ui"></a>使用 UI 编辑群集策略

可以使用群集策略 UI 或[群集策略 API](../../dev-tools/api/latest/policies.md) 编辑群集策略。 若要使用 UI 编辑群集策略，请执行以下操作：

1. 单击“群集”图标 ![“群集”图标](../../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击“群集策略”选项卡。

   > [!div class="mx-imgBorder"]
   > ![创建策略](../../_static/images/admin-cluster-management/cluster-policy-create.png)

3. 单击一个策略名称。
4. 单击 **“编辑”** 。
5. 在“定义”选项卡中，编辑策略定义。
6. 单击“更新”  。

## <a name="delete-a-cluster-policy-using-the-ui"></a>使用 UI 删除群集策略

可以使用群集策略 UI 或[群集策略 API](../../dev-tools/api/latest/policies.md) 删除群集策略。 若要使用 UI 删除群集策略，请执行以下操作：

1. 单击“群集”图标 ![“群集”图标](../../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击“群集策略”选项卡。

   > [!div class="mx-imgBorder"]
   > ![创建策略](../../_static/images/admin-cluster-management/cluster-policy-create.png)

3. 单击一个策略名称。
4. 单击 **“删除”** 。
5. 单击“删除”进行确认。

## <a name="cluster-policy-definitions"></a><a id="cluster-policy-definitions"> </a><a id="policy-def"> </a>群集策略定义

“群集策略定义”是一个由策略定义集合组成的 JSON 文档。

### <a name="in-this-section"></a>本节内容：

* [策略定义](#policy-definitions)
* [策略元素](#policy-elements)
* [群集策略属性路径](#cluster-policy-attribute-paths)
* [群集策略虚拟属性路径](#cluster-policy-virtual-attribute-paths)
* [数组特性](#array-attributes)
* [群集策略示例](#cluster-policy-examples)

### <a name="policy-definitions"></a>策略定义

“策略定义”是“定义属性的路径字符串”和“限制类型”之间的映射  。
每个属性只能有一个限制。 路径特定于资源类型，应反映资源创建 API 属性名称。 如果资源创建使用嵌套属性，则路径应将嵌套属性名称与点连接起来。 策略未指定的任何属性都是无限制的。

```
interface Policy {
  [path: string]: PolicyElement
}
```

### <a name="policy-elements"></a>策略元素

策略元素指定给定属性所受的一种支持的限制类型，还可以选择指定默认值。
即使策略中的属性没有限制，也可以指定默认值。

```
type PolicyElement = FixedPolicy | ForbiddenPolicy | (LimitingPolicyBase & LimitingPolicy);
type LimitingPolicy = WhitelistPolicy | BlacklistPolicy | RegexPolicy | RangePolicy | UnlimitedPolicy;
```

本部分介绍策略类型：

* [固定策略](#fixed-policy)
* [禁止的策略](#forbidden-policy)
* [限制策略：通用字段](#limiting-policies-common-fields)
* [允许列表策略](#whitelist-policy)
* [阻止列表策略](#blacklist-policy)
* [正则表达式策略](#regex-policy)
* [范围策略](#range-policy)
* [无限制策略](#unlimited-policy)

#### <a name="fixed-policy"></a>固定策略

将值限制为指定的值。  数值和布尔值以外的属性值必须由字符串表示或必须可转换为字符串。
当 `hidden` 标志存在并设置为 `true` 时，可以选择在 UI 中隐藏该属性。 固定策略不能指定默认值。

```
interface FixedPolicy {
    type: "fixed";
    value: string | number | boolean;
    hidden?: boolean;
}
```

##### <a name="example"></a>示例

```json
{
  "spark_version": { "type": "fixed", "value": "6.2", "hidden": true }
}
```

#### <a name="forbidden-policy"></a>禁止的策略

对于可选属性，请阻止使用该属性。

```
interface ForbiddenPolicy {
    type: "forbidden";
}
```

##### <a name="example"></a>示例

```json
{
  "instance_pool_id": { "type": "forbidden" }
}
```

#### <a name="limiting-policies-common-fields"></a>限制策略：通用字段

在限制策略中，可以指定两个附加字段：

* `defaultValue` - 填充 UI 中的群集创建窗体的值。
* `isOptional` - 属性的限制策略使其成为必需项。 若要使属性成为可选项，请将 `isOptional` 字段设置为 true。

```
interface LimitedPolicyBase {
    defaultValue?: string | number | boolean;
    isOptional?: boolean;
}
```

##### <a name="example"></a>示例

```json
{
  "instance_pool_id": { "type": "unlimited", "isOptional": true, "defaultValue": "id1" }
}
```

此示例策略将“池”字段指定为默认值 `id1`但将其设为可选字段。

#### <a name="whitelist-policy"></a>允许列表策略

允许值的列表。

```
interface WhitelistPolicy {
  type: "whitelist";
  values: (string | number | boolean)[];
}
```

##### <a name="example"></a>示例

```json
{
  "spark_version":  { "type": "whitelist", "values": [ "6.2", "6.3" ] }
}
```

#### <a name="blacklist-policy"></a>阻止列表策略

禁止值的列表。 由于值必须完全匹配，因此当属性在值的表示方式上较为宽松时（例如允许前导空格和尾随空格），此策略可能无法达到预期效果。

```
interface BlacklistPolicy {
  type: "blacklist";
  values: (string | number | boolean)[];
}
```

##### <a name="example"></a>示例

```json
{
  "spark_version":  { "type": "blacklist", "values": [ "4.0" ] }
}
```

#### <a name="regex-policy"></a>正则表达式策略

将值限制为与正则表达式匹配的值。 为了安全起见，匹配时，正则表达式始终固定在字符串值的开头和结尾。

```
interface RegexPolicy {
  type: "regex";
  pattern: string;
}
```

##### <a name="example"></a>示例

```json
{
  "spark_version":  { "type": "regex", "value": "5\\.[3456].*" }
}
```

#### <a name="range-policy"></a>范围策略

将值限制在 `minValue` 和 `maxValue` 属性指定的范围内。 该值必须是一个十进制数。
数值限制必须表示为双浮点值。 若要指示缺少特定的限制，可以省略 `minValue` 和 `maxValue` 中的一个。

```
interface RangePolicy {
  type: "range";
  minValue?: number;
  maxValue?: number;
}
```

##### <a name="example"></a>示例

```json
{
  "num_workers":  { "type": "range", "maxValue": 10 }
}
```

#### <a name="unlimited-policy"></a>无限制策略

不定义值限制。 可以使用此策略类型使属性成为必需项或在 UI 中设置默认值。

```
interface UnlimitedPolicy {
  type: "unlimited";
}
```

##### <a name="example"></a>示例

要求添加 `COST_BUCKET` 标记：

```json
{
  "custom_tags.COST_BUCKET":  { "type": "unlimited" }
}
```

为 Spark 配置变量设置默认值，同时允许省略（删除）该值：

```json
{
  "spark_conf.spark.my.conf":  { "type": "unlimited", "isOptional": true, "defaultValue": "my_value" }
}
```

### <a name="cluster-policy-attribute-paths"></a>群集策略属性路径

下表列出了支持的群集策略属性路径。

| 属性路径                                                       | 类型                              | 说明                                                                                                                     |
|----------------------------------------------------------------------|-----------------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| `cluster_name`                                                       | string                            | 群集名称。                                                                                                               |
| `spark_conf.*`                                                       | 可选字符串                   | 通过追加配置键名称来控制特定的配置值。 例如，`spark_conf.spark.executor.memory`。 |
| `instance_pool_id`                                                   | string                            | 隐藏后，从 UI 中删除池选择。                                                                                |
| `num_workers`                                                        | 可选数值                   | 隐藏后，从 UI 中删除辅助角色数目规范。                                                               |
| `autoscale.min_workers`                                              | 可选数值                   | 隐藏后，从 UI 中删除“最小辅助角色数目”字段。                                                               |
| `autoscale.max_workers`                                              | 可选数值                   | 隐藏后，从 UI 中删除“最大辅助角色数目”字段。                                                               |
| `autotermination_minutes`                                            | number                            | 值 0 表示没有自动终止。 隐藏后，从 UI 中删除自动终止复选框和值输入。          |
| `node_type_id`                                                       | string                            | 隐藏后，从 UI 中删除工作器节点类型选择。                                                                |
| `driver_node_type_id`                                                | 可选字符串                   | 隐藏后，从 UI 中删除驱动器节点类型选择。                                                                |
| `custom_tags.*`                                                      | string                            | 通过追加标记名来控制特定的标记值。 例如，`custom_tags.<mytag>`。                                      |
| `spark_version`                                                      | string                            | Spark 映像版本名称（通过 API 指定）。                                                                                |
| `docker_image.url`                                                   | string                            | 控制 Databricks 容器服务映像 URL。 隐藏后，从 UI 中删除“Databricks 容器服务”部分。   |
| `docker_image.basic_auth.username`                                   | string                            | 用于 Databricks 容器服务映像基本身份验证的用户名。                                                      |
| `docker_image.basic_auth.password`                                   | string                            | 用于 Databricks 容器服务映像基本身份验证的密码。                                                      |
| `cluster_log_conf.type`                                              | string                            | DBFS                                                                                                                            |
| `cluster_log_conf.path`                                              | string                            | 日志文件的目标 URL。                                                                                               |
| `single_user_name`                                                   | string                            | 用于凭据直通单用户访问的用户名。                                                                        |
| `init_scripts.*.dbfs.destination`, `init_scripts.*.file.destination` | string                            | `*` 是指属性数组中初始化脚本的索引，请参阅[数组属性](#array-attributes)。                   |

### <a name="cluster-policy-virtual-attribute-paths"></a>群集策略虚拟属性路径

| 属性路径                    | 类型                              | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|-----------------------------------|-----------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `dbus_per_hour`                   | 数字                            | 计算的属性，表示包括驱动程序节点在内的群集的 DBU 开销（在自动缩放群集的情况下为最大值）。 与范围限制配合使用。                                                                                                                                                                                                                                                                                                            |
| `cluster_type`                    | string                            | 表示可以创建的群集类型：<br><br>* `all-purpose` 用于 Azure Databricks 通用群集<br>* `job` 用于作业计划程序创建的作业群集<br><br>将根据策略创建的某些类型的群集加入允许列表或阻止列表。 如果不允许使用 `all-purpose` 值，则在通用群集创建窗体中不会显示该策略。 如果不允许使用 `job` 值，则在作业新群集窗体中不会显示该策略。 |

### <a name="array-attributes"></a>数组特性

可以通过两种方式指定数组属性的策略：

* 对所有数组元素采用一般限制。 这些限制在策略路径中使用 `*` 通配符。
* 对特定索引处的数组元素采用特定限制。 这些限制在路径中使用数字。

例如，在数组属性 `init_scripts` 中，一般路径以 `init_scripts.*` 开头，特定路径以 `init_scripts.<n>` 开头，其中 `<n>` 是数组中的整数索引（从 0 开始）。
可以将一般限制和特定限制结合起来，在这种情况下，一般限制适用于没有特定限制的每个数组元素。 在每种情况下，只有一种政策限制适用。

阵列策略的典型用例有：

* 需要特定于包含内容的条目。 例如：

  ```json
  {
    "init_scripts.0.dbfs.destination": {
      "type": "fixed",
      "value": "<required-script-1>"
    },
    "init_scripts.1.dbfs.destination": {
      "type": "fixed",
      "value": "<required-script-2>"
    }
  }
  ```

  如果不指定顺序，则不能要求特定值。

* 需要整个列表的固定值。 例如：

  ```json
  {
    "init_scripts.0.dbfs.destination": {
      "type": "fixed",
      "value": "<required-script-1>"
    },
    "init_scripts.*.dbfs.destination": {
      "type": "forbidden"
    }
  }
  ```

* 禁止完全使用。

  ```json
  {
    "init_scripts.*.dbfs.destination": {
      "type": "forbidden"
    }
  }
  ```

* 允许任意数量的条目，但仅限于特定限制。 例如：

  ```json
  {
     "init_scripts.*.dbfs.destination": {
      "type": "regex",
      "pattern": ".*<required-content>.*"
    }
  }
  ```

如果是 `init_scripts` 路径，数组可以包含多个结构中的一个，根据用例的不同，可能需要对这些结构的所有可能变体进行处理。 例如，若要要求一组特定的初始化脚本，并且不允许其他版本的任何变体，可以使用以下模式：

```json
{
  "init_scripts.1.dbfs.destination": {
    "type": "fixed",
    "value": "dbfs://<dbfs-path>"
  },
  "init_scripts.*.dbfs.destination": {
    "type": "forbidden"
  },
  "init_scripts.*.file.destination": {
    "type": "forbidden"
  },
}
```

### <a name="cluster-policy-examples"></a>群集策略示例

#### <a name="in-this-section"></a>本节内容：

* [常规群集策略](#general-cluster-policy)
* [简单的中型策略](#simple-medium-sized-policy)
* [仅作业策略](#job-only-policy)
* [单节点策略](#single-node-policy)
* [高并发传递策略](#high-concurrency-passthrough-policy)
* [外部元存储策略](#external-metastore-policy)

#### <a name="general-cluster-policy"></a>常规群集策略

常规用途群集策略，用于指导用户并限制某些功能，同时要求使用标记、限制实例的最大数目并强制执行超时。

```json
{
  "spark_conf.spark.databricks.cluster.profile": {
    "type": "fixed",
    "value": "serverless",
    "hidden": true
  },
  "instance_pool_id": {
    "type": "forbidden",
    "hidden": true
  },
  "spark_version": {
    "type": "regex",
    "pattern": "6\\.[0-9]+\\.x-scala.*"
  },
  "node_type_id": {
    "type": "whitelist",
    "values": [
      "Standard_L4s",
      "Standard_L8s",
      "Standard_L16s"
    ],
    "defaultValue": "Standard_L16s_v2"
  },
  "driver_node_type_id": {
    "type": "fixed",
    "value": "Standard_L16s_v2",
    "hidden": true
  },
  "autoscale.min_workers": {
    "type": "fixed",
    "value": 1,
    "hidden": true
  },
  "autoscale.max_workers": {
    "type": "range",
    "maxValue": 25,
    "defaultValue": 5
  },
  "autotermination_minutes": {
    "type": "fixed",
    "value": 30,
    "hidden": true
  },
  "custom_tags.team": {
    "type": "fixed",
    "value": "product"
  }
}
```

#### <a name="simple-medium-sized-policy"></a>简单的中型策略

允许用户使用最小配置创建中型群集。 创建时唯一的必需字段是群集名称；其余为固定字段和隐藏字段。

```json
{
  "instance_pool_id": {
    "type": "forbidden",
    "hidden": true
  },
  "spark_conf.spark.databricks.cluster.profile": {
    "type": "forbidden",
    "hidden": true
  },
  "autoscale.min_workers": {
    "type": "fixed",
    "value": 1,
    "hidden": true
  },
  "autoscale.max_workers": {
    "type": "fixed",
    "value": 10,
    "hidden": true
  },
  "autotermination_minutes": {
    "type": "fixed",
    "value": 60,
    "hidden": true
  },
  "node_type_id": {
    "type": "fixed",
    "value": "Standard_L8s_v2",
    "hidden": true
  },
  "driver_node_type_id": {
    "type": "fixed",
    "value": "Standard_L8s_v2",
    "hidden": true
  },
  "spark_version": {
    "type": "fixed",
    "value": "7.x-scala2.11",
    "hidden": true
  },
  "custom_tags.team": {
    "type": "fixed",
    "value": "product"
  }
}
```

#### <a name="job-only-policy"></a>仅作业策略

允许用户创建作业群集并使用群集运行作业。 用户无法使用此策略创建通用群集。

```json
{
  "cluster_type": {
    "type": "fixed",
    "value": "job"
  },
  "dbus_per_hour": {
    "type": "range",
    "maxValue": 100
  },
  "instance_pool_id": {
    "type": "forbidden",
    "hidden": true
  },
  "num_workers": {
    "type": "range",
    "minValue": 1
  },
  "node_type_id": {
    "type": "regex",
    "pattern": "Standard_[DLS]*[1-6]{1,2}_v[2,3]"
  },
  "driver_node_type_id": {
    "type": "regex",
    "pattern": "Standard_[DLS]*[1-6]{1,2}_v[2,3]"
  },
  "spark_version": {
    "type": "regex",
    "pattern": "6\\.[0-9]+\\.x-scala.*"
  },
  "custom_tags.team": {
    "type": "fixed",
    "value": "product"
  }
}
```

#### <a name="single-node-policy"></a>单节点策略

允许用户创建不启用辅助角色和 Spark 的单节点群集。 可以在[单节点群集策略](../../clusters/single-node.md#single-node-policy)找到示例单节点策略。

#### <a name="high-concurrency-passthrough-policy"></a>高并发传递策略

允许用户在高并发模式下创建默认启用了传递的群集。 这简化了管理员的设置，因为用户需要手动设置相应的 Spark 参数。

```json
{
  "spark_conf.spark.databricks.passthrough.enabled": {
    "type": "fixed",
    "value": "true"
  },
  "spark_conf.spark.databricks.repl.allowedLanguages": {
    "type": "fixed",
    "value": "python,sql"
  },
  "spark_conf.spark.databricks.cluster.profile": {
    "type": "fixed",
    "value": "serverless"
  },
  "spark_conf.spark.databricks.pyspark.enableProcessIsolation": {
    "type": "fixed",
    "value": "true"
  },
  "custom_tags.ResourceClass": {
    "type": "fixed",
    "value": "Serverless"
  }
}
```

#### <a name="external-metastore-policy"></a>外部元存储策略

允许用户创建已附加管理员定义的元存储的群集。 这可用于允许用户创建自己的群集，且无需其他配置。

```json
{
  "spark_conf.spark.hadoop.javax.jdo.option.ConnectionURL": {
      "type": "fixed",
      "value": "jdbc:sqlserver://<jdbc-url>"
  },
  "spark_conf.spark.hadoop.javax.jdo.option.ConnectionDriverName": {
      "type": "fixed",
      "value": "com.microsoft.sqlserver.jdbc.SQLServerDriver"
  },
  "spark_conf.spark.databricks.delta.preview.enabled": {
      "type": "fixed",
      "value": "true"
  },
  "spark_conf.spark.hadoop.javax.jdo.option.ConnectionUserName": {
      "type": "fixed",
      "value": "<metastore-user>"
  },
  "spark_conf.spark.hadoop.javax.jdo.option.ConnectionPassword": {
      "type": "fixed",
      "value": "<metastore-password>"
  }
}
```