---
title: Python 插件 - Azure 数据资源管理库
description: 本文介绍 Azure 数据资源管理器中的 Python 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: reference
origin.date: 04/01/2020
ms.date: 08/18/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: a469b646d93dc9915642e280a0a54b69a9f8cddb
ms.sourcegitcommit: 39410f3ed7bdeafa1099ba5e9ec314b4255766df
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2020
ms.locfileid: "90678411"
---
# <a name="python-plugin"></a>Python 插件

::: zone pivot="azuredataexplorer"

Python 插件使用 Python 脚本运行用户定义函数 (UDF)。 Python 脚本获取表格数据作为输入，并预期生成一个表格输出。
插件的运行时托管在[沙盒](../concepts/sandboxes.md)中，运行在群集的节点上。

## <a name="syntax"></a>语法

*T* `|` `evaluate` [`hint.distribution` `=` (`single` | `per_node`)] `python(`*output_schema*`,` *script* [`,` *script_parameters*][`,` *external_artifacts*]`)`

## <a name="arguments"></a>参数

* *output_schema*：`type` 文本，定义由 Python 代码返回的表格数据的输出模式。
    * 格式为：`typeof(`*ColumnName*`:` *ColumnType*[, ...]`)`.例如， `typeof(col1:string, col2:long)`。
    * 若要扩展输入架构，请使用以下语法：`typeof(*, col1:string, col2:long)`
* *script*：`string` 文本，是要执行的有效 Python 脚本。
* *script_parameters*：可选的 `dynamic` 文本。 它是一个名称/值对的属性包，作为保留的 `kargs` 字典传递给 Python 脚本。 有关详细信息，请参阅[保留 Python 变量](#reserved-python-variables)。
* *hint.distribution*：一个可选的提示，用于在多个群集节点上分布插件的执行。
  * 默认值为 `single`。
  * `single`：脚本的单个实例将对查询数据整体运行。
  * `per_node`：如果在分发 Python 块之前执行查询，则脚本的一个实例将在每个节点上对其包含的数据运行。
* *external_artifacts*：可选的 `dynamic` 文本，是名称和 URL 对的属性包，用于可从云存储访问的项目。 它们可供脚本在运行时使用。
  * 此属性包中引用的 URL 必须是：
    * 包含在群集的[标注策略](../management/calloutpolicy.md)中。
    * 在公开的位置，或提供必要的凭据，如[存储连接字符串](../api/connection-strings/storage.md)中所述。
  * 这些项目可供脚本从本地临时目录 `.\Temp` 使用。 属性包中提供的名称用作本地文件名。 请参阅[示例](#examples)。
  * 有关详细信息，请参阅[安装 Python 插件的包](#install-packages-for-the-python-plugin)。 

## <a name="reserved-python-variables"></a>保留 Python 变量

以下变量是为 Kusto 查询语言和 Python 代码之间的交互而保留的。

* `df`：作为 `pandas` 数据帧的输入表格数据（上述 `T` 的值）。
* `kargs`：作为 Python 字典的 script_parameters 参数的值。
* `result`：由 Python 脚本创建的 `pandas` 数据帧，其值成为发送到插件后面的 Kusto 查询操作符的表格数据。

## <a name="enable-the-plugin"></a>启用插件

* 默认禁用该插件。
* 若要启用该插件，请参阅[先决条件](../concepts/sandboxes.md#prerequisites)列表。
* 在 Azure 门户的群集的[“配置”选项卡](../../language-extensions.md)中启用或禁用插件。

## <a name="python-sandbox-image"></a>Python 沙盒映像

* Python 沙盒映像基于 Anaconda 5.2.0 发行版和 Python 3.6 引擎 。
  请参阅 [Anaconda 包](https://docs.anaconda.com/anaconda/packages/old-pkg-lists/5.2.0/py3.6_win-64/)的列表。
  
  > [!NOTE]
  > 有一小部分包可能与运行插件的沙盒所实施的限制不兼容。
  
* Python 映像还包含常见 ML 包：`tensorflow`、`keras`、`torch`、`hdbscan`、`xgboost` 和其他有用的包。
* 插件默认导入 numpy（作为 `np`）和 pandas（作为 `pd`） 。  你可以根据需要导入其他模块。

## <a name="use-ingestion-from-query-and-update-policy"></a>使用来自查询和更新策略的引入

* 在以下查询中使用插件：
  * 定义为[更新策略](../management/updatepolicy.md)的一部分，其源表引入为使用非流式处理引入。
  * 作为[从查询引入](../management/data-ingestion/ingest-from-query.md)（例如 `.set-or-append`）的命令的一部分运行。
    在这两种情况下，都请验证引入的量和频率，以及 Python 逻辑使用的复杂性和资源是否与[沙盒限制](../concepts/sandboxes.md#limitations)和群集的可用资源一致。 否则，可能会导致[限制错误](../concepts/sandboxes.md#errors)。
* 不能在定义为更新策略一部分的查询中使用插件，该策略的源表是使用[流式处理引入](../../ingest-data-streaming.md)引入的。

## <a name="examples"></a>示例

```kusto
range x from 1 to 360 step 1
| evaluate python(
//
typeof(*, fx:double),               //  Output schema: append a new fx column to original table 
//
'result = df\n'                     //  The Python decorated script
'n = df.shape[0]\n'
'g = kargs["gain"]\n'
'f = kargs["cycles"]\n'
'result["fx"] = g * np.sin(df["x"]/n*2*np.pi*f)\n'
//
, pack('gain', 100, 'cycles', 4)    //  dictionary of parameters
)
| render linechart 
```

:::image type="content" source="images/plugin/sine-demo.png" alt-text="正弦演示" border="false":::

```kusto
print "This is an example for using 'external_artifacts'"
| evaluate python(
    typeof(File:string, Size:string),
    "import os\n"
    "result = pd.DataFrame(columns=['File','Size'])\n"
    "sizes = []\n"
    "path = '.\\\\Temp'\n"
    "files = os.listdir(path)\n"
    "result['File']=files\n"
    "for file in files:\n"
    "    sizes.append(os.path.getsize(path + '\\\\' + file))\n"
    "result['Size'] = sizes\n"
    "\n",
    external_artifacts = 
        dynamic({"this_is_my_first_file":"https://kustoscriptsamples.blob.core.chinacloudapi.cn/samples/R/sample_script.r",
                 "this_is_a_script":"https://kustoscriptsamples.blob.core.chinacloudapi.cn/samples/python/sample_script.py"})
)
```

| 文件                  | 大小 |
|-----------------------|------|
| this_is_a_script      | 120  |
| this_is_my_first_file | 105  |

## <a name="performance-tips"></a>性能提示

* 将插件的输入数据集减少到所需的最小量（列/行）。
    * 在可能的情况下，通过 Kusto 的查询语言对源数据集使用筛选器。
    * 若要对源列的子集进行计算，请在调用插件之前仅投影这些列。
* 如果脚本中的逻辑可分发，请使用 `hint.distribution = per_node`。
    * 还可以使用[分区运算符](partitionoperator.md)对输入数据集进行分区。
* 尽可能使用 Kusto 的查询语言来实现 Python 脚本的逻辑。

    ## <a name="example"></a>示例

    ```kusto    
    .show operations
    | where StartedOn > ago(7d) // Filtering out irrelevant records before invoking the plugin
    | project d_seconds = Duration / 1s // Projecting only a subset of the necessary columns
    | evaluate hint.distribution = per_node python( // Using per_node distribution, as the script's logic allows it
        typeof(*, _2d:double),
        'result = df\n'
        'result["_2d"] = 2 * df["d_seconds"]\n' // Negative example: this logic should have been written using Kusto's query language
      )
    | summarize avg = avg(_2d)
    ```

## <a name="usage-tips"></a>使用提示

* 若要在 `Kusto.Explorer` 中生成包含 Python 脚本的多行字符串，请从你最喜欢的 Python 编辑器（Jupyter、Visual Studio Code、PyCharm 等）中复制 Python 脚本  。 
  现在执行以下操作之一：
    * 按“F2”打开“以 Python 编辑”窗口。 将脚本粘贴到此窗口。 选择“确定”。 脚本将用引号和新行修饰，使其在 Kusto 中有效，并自动粘贴到查询选项卡。
    * 将 Python 代码直接粘贴到查询选项。选择这些行，然后按 Ctrl+K、Ctrl+S 热键，如上所示装饰它们 。 若要反转，请按 Ctrl+K、Ctrl+M 热键 。 请参阅[查询编辑器快捷方式](../tools/kusto-explorer-shortcuts.md#query-editor)的完整列表。
* 若要避免 Kusto 字符串分隔符和 Python 字符串文本之间的冲突，请使用：
     * Kusto 查询中 Kusto 字符串文本的单引号字符 (`'`)
     * Python 脚本中用于 Python 字符串的双引号字符 (`"`)
* 使用 [`externaldata` 运算符](externaldata-operator.md)获取存储在外部位置（例如 Azure Blob 存储）中的脚本内容。
  
    ## <a name="example"></a>示例

    ```kusto
    let script = 
        externaldata(script:string)
        [h'https://kustoscriptsamples.blob.core.chinacloudapi.cn/samples/python/sample_script.py']
        with(format = raw);
    range x from 1 to 360 step 1
    | evaluate python(
        typeof(*, fx:double),
        toscalar(script), 
        pack('gain', 100, 'cycles', 4))
    | render linechart 
    ```

## <a name="install-packages-for-the-python-plugin"></a>安装 Python 插件的包

由于以下原因，你可能需要自行安装包：

* 包是专用的，是你自己的。
* 包是公共的，但不包括在插件的基本映像中。

安装包，如下所示：

### <a name="prerequisites"></a>先决条件

  1. 创建 blob 容器来承载包，最好与群集位于同一个位置。 例如 `https://chinaeast2.blob.core.chinacloudapi.cn/python`，假设群集位于中国东部。
  1. 更改群集的[标注策略](../management/calloutpolicy.md)以允许访问该位置。
        * 此更改需要 [AllDatabasesAdmin](../management/access-control/role-based-authorization.md) 权限。

        * 例如，若要启用对位于 `https://chinaeast2.blob.core.chinacloudapi.cn/python` 中的 blob 的访问，请运行以下命令：

        ```kusto
        .alter-merge cluster policy callout @'[ { "CalloutType": "sandbox_artifacts", "CalloutUriRegex": "chinaeast2\\.blob\\.core\\.windows\\.net/python/","CanCall": true } ]'
        ```

### <a name="install-packages"></a>安装包

1. 对于 [PyPi](https://pypi.org/) 或其他通道中的公共包，请下载包及其依赖项。

   * 如果需要，编译滚轮 (`*.whl`) 文件。
   * 从本地 Python 环境中的 cmd 窗口运行：
    
    ```python
    pip wheel [-w download-dir] package-name.
    ```

1. 创建一个 zip 文件，其中包含所需的包及其依赖项。

    * 对于专用包：压缩包的文件夹及其依赖项的文件夹。
    * 对于公共包，对在上一步中下载的文件进行压缩。
    
    > [!NOTE]
    > * 请确保压缩 `.whl` 文件本身，而不是其父文件夹。
    > * 对于基本沙箱映像中已经存在的具有相同版本的包，你可以跳过 `.whl` 文件。

1. 将压缩文件上传到项目位置中的 blob（从步骤 1 开始）。

1. 调用 `python` 插件。
    * 使用包含名称和对 zip 文件（blob 的 URL）的引用的属性包指定 `external_artifacts` 参数。
    * 在内联 python 代码中，从 `sandbox_utils` 导入 `Zipackage`，并使用 zip 文件的名称调用其 `install()` 方法。

### <a name="example"></a>示例

安装生成伪数据的 [Faker](https://pypi.org/project/Faker/) 包。

```kusto
range ID from 1 to 3 step 1 
| extend Name=''
| evaluate python(typeof(*),
    'from sandbox_utils import Zipackage\n'
    'Zipackage.install("Faker.zip")\n'
    'from faker import Faker\n'
    'fake = Faker()\n'
    'result = df\n'
    'for i in range(df.shape[0]):\n'
    '    result.loc[i, "Name"] = fake.name()\n',
    external_artifacts=pack('faker.zip', 'https://artifacts.blob.core.chinacloudapi.cn/kusto/Faker.zip?...'))
```

| ID | 名称         |
|----|--------------|
|   1| Gary Tapia   |
|   2| Emma Evans   |
|   3| Ashley Bowen |

---

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
