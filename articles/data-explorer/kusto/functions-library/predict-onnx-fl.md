---
title: predict_onnx_fl() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的用户定义函数 predict_onnx_fl()。
author: orspod
ms.author: v-tawe
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: reference
origin.date: 09/09/2020
ms.date: 10/29/2020
ms.openlocfilehash: e3c3ef5959702f55d48965e0b222cc8f82bbbdcd
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105531"
---
# <a name="predict_onnx_fl"></a>predict_onnx_fl()

函数 `predict_onnx_fl()` 使用现有的已定型机器学习模型进行预测。 此模型已转换为 [ONNX](https://onnx.ai/) 格式，已序列化为字符串，并已保存在标准的 Azure 数据资源管理器表中。

> [!NOTE]
> * `predict_onnx_fl()` 是 [UDF（用户定义的函数）](../query/functions/user-defined-functions.md)。
> * 此函数包含内联 Python，需要在群集上[启用 python() 插件](../query/pythonplugin.md#enable-the-plugin)。 有关详细信息，请参阅[用法](#usage)。

## <a name="syntax"></a>语法

`T | invoke predict_onnx_fl(`*models_tbl*`,` *model_name*`,` *features_cols*`,` *pred_col*`)`

## <a name="arguments"></a>参数

* models_tbl：包含所有序列化模型的表的名称。 此表必须包含以下列：
    * name：模型名称
    * timestamp：模型训练时间
    * model：序列化模型的字符串表示形式
* model_name：要使用的特定模型的名称。
* features_cols：动态数组，其中包含供模型用来预测的特征列的名称。
* pred_col：存储预测的列的名称。

## <a name="usage"></a>使用情况

`predict_onnx_fl()` 是用户定义的[表格函数](../query/functions/user-defined-functions.md#tabular-function)，将使用 [invoke 运算符](../query/invokeoperator.md)来应用。 可以在查询中嵌入其代码，或将其安装在数据库中。 用法选项有两种：临时使用和永久使用。 有关示例，请参阅下面的选项卡。

# <a name="ad-hoc"></a>[临时](#tab/adhoc)

对于临时使用，请使用 [let 语句](../query/letstatement.md)嵌入代码。 不需要权限。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let predict_onnx_fl=(samples:(*), models_tbl:(name:string, timestamp:datetime, model:string), model_name:string, features_cols:dynamic, pred_col:string)
{
    let model_str = toscalar(models_tbl | where name == model_name | top 1 by timestamp desc | project model);
    let kwargs = pack('smodel', model_str, 'features_cols', features_cols, 'pred_col', pred_col);
    let code =
    '\n'
    'import pickle\n'
    'import binascii\n'
    '\n'
    'smodel = kargs["smodel"]\n'
    'features_cols = kargs["features_cols"]\n'
    'pred_col = kargs["pred_col"]\n'
    'bmodel = binascii.unhexlify(smodel)\n'
    '\n'
    'features_cols = kargs["features_cols"]\n'
    'pred_col = kargs["pred_col"]\n'
    '\n'
    'import onnxruntime as rt\n'
    'sess = rt.InferenceSession(bmodel)\n'
    'input_name = sess.get_inputs()[0].name\n'
    'label_name = sess.get_outputs()[0].name\n'
    'df1 = df[features_cols]\n'
    'predictions = sess.run([label_name], {input_name: df1.values.astype(np.float32)})[0]\n'
    '\n'
    'result = df\n'
    'result[pred_col] = pd.DataFrame(predictions, columns=[pred_col])'
    '\n'
    ;
    samples | evaluate python(typeof(*), code, kwargs)
};
//
// Predicts room occupancy from sensors measurements, and calculates the confusion matrix
//
// Occupancy Detection is an open dataset from UCI Repository at https://archive.ics.uci.edu/ml/datasets/Occupancy+Detection+
// It contains experimental data for binary classification of room occupancy from Temperature,Humidity,Light and CO2.
// Ground-truth labels were obtained from time stamped pictures that were taken every minute
//
OccupancyDetection 
| where Test == 1
| extend pred_Occupancy=bool(0)
| invoke predict_onnx_fl(ML_Models, 'ONNX-Occupancy', pack_array('Temperature', 'Humidity', 'Light', 'CO2', 'HumidityRatio'), 'pred_Occupancy')
| summarize n=count() by Occupancy, pred_Occupancy
```

# <a name="persistent"></a>[Persistent](#tab/persistent)

如果是永久使用，请使用 [.create 函数](../management/create-function.md)。 创建函数需要[数据库用户权限](../management/access-control/role-based-authorization.md)。

### <a name="one-time-installation"></a>一次性安装

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
.create-or-alter function with (folder = "Packages\\ML", docstring = "Predict using ONNX model")
predict_onnx_fl(samples:(*), models_tbl:(name:string, timestamp:datetime, model:string), model_name:string, features_cols:dynamic, pred_col:string)
{
    let model_str = toscalar(models_tbl | where name == model_name | top 1 by timestamp desc | project model);
    let kwargs = pack('smodel', model_str, 'features_cols', features_cols, 'pred_col', pred_col);
    let code =
    '\n'
    'import pickle\n'
    'import binascii\n'
    '\n'
    'smodel = kargs["smodel"]\n'
    'features_cols = kargs["features_cols"]\n'
    'pred_col = kargs["pred_col"]\n'
    'bmodel = binascii.unhexlify(smodel)\n'
    '\n'
    'features_cols = kargs["features_cols"]\n'
    'pred_col = kargs["pred_col"]\n'
    '\n'
    'import onnxruntime as rt\n'
    'sess = rt.InferenceSession(bmodel)\n'
    'input_name = sess.get_inputs()[0].name\n'
    'label_name = sess.get_outputs()[0].name\n'
    'df1 = df[features_cols]\n'
    'predictions = sess.run([label_name], {input_name: df1.values.astype(np.float32)})[0]\n'
    '\n'
    'result = df\n'
    'result[pred_col] = pd.DataFrame(predictions, columns=[pred_col])'
    '\n'
    ;
    samples | evaluate python(typeof(*), code, kwargs)
}
```

### <a name="usage"></a>使用情况

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
//
// Predicts room occupancy from sensors measurements, and calculates the confusion matrix
//
// Occupancy Detection is an open dataset from UCI Repository at https://archive.ics.uci.edu/ml/datasets/Occupancy+Detection+
// It contains experimental data for binary classification of room occupancy from Temperature,Humidity,Light and CO2.
// Ground-truth labels were obtained from time stamped pictures that were taken every minute
//
OccupancyDetection 
| where Test == 1
| extend pred_Occupancy=bool(0)
| invoke predict_onnx_fl(ML_Models, 'ONNX-Occupancy', pack_array('Temperature', 'Humidity', 'Light', 'CO2', 'HumidityRatio'), 'pred_Occupancy')
| summarize n=count() by Occupancy, pred_Occupancy
```

---

混淆矩阵：
<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
Occupancy   pred_Occupancy  n
TRUE        TRUE            3006
FALSE       TRUE            112
TRUE        FALSE           15
FALSE       FALSE           9284
```
