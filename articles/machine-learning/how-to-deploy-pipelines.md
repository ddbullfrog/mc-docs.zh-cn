---
title: 发布 ML 管道
titleSuffix: Azure Machine Learning
description: 利用机器学习管道和适用于 Python 的 Azure 机器学习 SDK 运行机器学习工作流。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: sgilley
ms.author: laobri
author: lobrien
ms.date: 8/25/2020
ms.topic: conceptual
ms.custom: how-to, contperfq1
ms.openlocfilehash: 99c4151924e8844212ed0b8d394f6cccf3234dfd
ms.sourcegitcommit: 7320277f4d3c63c0b1ae31ba047e31bf2fe26bc6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92117963"
---
# <a name="publish-and-track-machine-learning-pipelines"></a>发布和跟踪机器学习管道



本文将演示如何与同事或客户共享机器学习管道。

机器学习管道是用于机器学习任务的可重用工作流。 管道的一个益处是加强了协作。 你也可以对管道进行版本管理，这样客户就可以在你使用新版本的同时使用当前的模型。 

## <a name="prerequisites"></a>必备条件

* 创建用于保留所有管道资源的 [Azure 机器学习工作区](how-to-manage-workspace.md)

* [配置开发环境](how-to-configure-environment.md)以安装 Azure 机器学习 SDK，或使用已经安装了该 SDK 的 [Azure 机器学习计算实例](concept-compute-instance.md)

* 创建并运行机器学习管道，例如，按照[教程：生成 Azure 机器学习管道以用于批量评分](tutorial-pipeline-batch-scoring-classification.md)中的说明操作。 对于其他选项，请参阅[使用 Azure 机器学习 SDK 创建并运行机器学习管道](how-to-create-your-first-pipeline.md)

## <a name="publish-a-pipeline"></a>发布管道

启动并运行管道之后，你可以发布管道，以便它使用其他输入运行。 若要使已发布的管道的 REST 终结点接受参数，必须将管道配置为对各有差异的参数使用 `PipelineParameter` 对象。

1. 若要创建管道参数，请使用带默认值的 [PipelineParameter](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.pipelineparameter?view=azure-ml-py&preserve-view=true) 对象。

   ```python
   from azureml.pipeline.core.graph import PipelineParameter
   
   pipeline_param = PipelineParameter(
     name="pipeline_arg",
     default_value=10)
   ```

2. 按如下所示，将此 `PipelineParameter` 对象作为参数添加到管道中的任一步骤：

   ```python
   compareStep = PythonScriptStep(
     script_name="compare.py",
     arguments=["--comp_data1", comp_data1, "--comp_data2", comp_data2, "--output_data", out_data3, "--param1", pipeline_param],
     inputs=[ comp_data1, comp_data2],
     outputs=[out_data3],
     compute_target=compute_target,
     source_directory=project_folder)
   ```

3. 发布此管道，调用时它会接受参数。

   ```python
   published_pipeline1 = pipeline_run1.publish_pipeline(
        name="My_Published_Pipeline",
        description="My Published Pipeline Description",
        version="1.0")
   ```

## <a name="run-a-published-pipeline"></a>运行已发布的管道

所有已发布的管道都具有 REST 终结点。 使用管道终结点，可以从任何外部系统（包括非 Python 客户端）触发管道运行。 在批量评分和重新训练方案中，此终结点支持“托管可重复性”。

若要调用上述管道的运行，需要 Azure Active Directory 身份验证标头令牌。 [AzureCliAuthentication 类](https://docs.microsoft.com/python/api/azureml-core/azureml.core.authentication.azurecliauthentication?view=azure-ml-py&preserve-view=true)参考和 [Azure 机器学习中的身份验证](https://aka.ms/pl-restep-auth)笔记本中介绍了如何获取这样的令牌。

```python
from azureml.pipeline.core import PublishedPipeline
import requests

response = requests.post(published_pipeline1.endpoint,
                         headers=aad_token,
                         json={"ExperimentName": "My_Pipeline",
                               "ParameterAssignments": {"pipeline_arg": 20}})
```

对于 `ParameterAssignments` 键，POST 请求的 `json` 参数必须包含一个具有管道参数及其值的字典。 此外，`json` 参数可能包含以下键：

| 键 | 说明 |
| --- | --- | 
| `ExperimentName` | 与此终结点关联的试验的名称 |
| `Description` | 描述终结点的自由格式文本 | 
| `Tags` | 可用于标记和注释请求的自由格式键值对  |
| `DataSetDefinitionValueAssignments` | 用于在不重新训练的情况下更改数据集的字典（请参阅以下讨论） | 
| `DataPathAssignments` | 用于在不重新训练的情况下更改数据路径的字典（请参阅以下讨论） | 

### <a name="changing-datasets-and-datapaths-without-retraining"></a>在不重新训练的情况下更改数据集和数据路径

你可能想要对其他数据集和数据路径进行训练和推理。 例如，你可能想要对较小的稀疏数据集进行训练，对完整数据集进行推理。 可使用请求的 `json` 参数中的 `DataSetDefinitionValueAssignments` 键切换数据集。 可使用 `DataPathAssignments` 切换数据路径。 两者的方法类似：

1. 在管道定义脚本中，为数据集创建 `PipelineParameter`。 在 `PipelineParameter` 中创建 `DatasetConsumptionConfig` 或 `DataPath`：

    ```python
    tabular_dataset = Dataset.Tabular.from_delimited_files('https://dprepdata.blob.core.windows.net/demo/Titanic.csv')
    tabular_pipeline_param = PipelineParameter(name="tabular_ds_param", default_value=tabular_dataset)
    tabular_ds_consumption = DatasetConsumptionConfig("tabular_dataset", tabular_pipeline_param)
    ```

1. 在 ML 脚本中，使用 `Run.get_context().input_datasets` 访问动态指定的数据集：

    ```python
    from azureml.core import Run
    
    input_tabular_ds = Run.get_context().input_datasets['tabular_dataset']
    dataframe = input_tabular_ds.to_pandas_dataframe()
    # ... etc ...
    ```

    请注意，ML 脚本访问为 `DatasetConsumptionConfig` 指定的值 (`tabular_dataset`)，不访问 `PipelineParameter` 的值 (`tabular_ds_param`)。

1. 在管道定义脚本中，将 `DatasetConsumptionConfig` 设置为 `PipelineScriptStep` 的参数：

    ```python
    train_step = PythonScriptStep(
        name="train_step",
        script_name="train_with_dataset.py",
        arguments=["--param1", tabular_ds_consumption],
        inputs=[tabular_ds_consumption],
        compute_target=compute_target,
        source_directory=source_directory)
    
    pipeline = Pipeline(workspace=ws, steps=[train_step])
    ```

1. 若要在推理 REST 调用中动态切换数据集，请使用 `DataSetDefinitionValueAssignments`：
    
    ```python
    tabular_ds1 = Dataset.Tabular.from_delimited_files('path_to_training_dataset')
    tabular_ds2 = Dataset.Tabular.from_delimited_files('path_to_inference_dataset')
    ds1_id = tabular_ds1.id
    d22_id = tabular_ds2.id
    
    response = requests.post(rest_endpoint, 
                             headers=aad_token, 
                             json={
                                "ExperimentName": "MyRestPipeline",
                               "DataSetDefinitionValueAssignments": {
                                    "tabular_ds_param": {
                                        "SavedDataSetReference": {"Id": ds1_id #or ds2_id
                                    }}}})
    ```

有关此方法的完整示例，请参阅[展示数据集和 PipelineParameter](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-showcasing-dataset-and-pipelineparameter.ipynb) 和[展示数据路径和 PipelineParameter](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-showcasing-datapath-and-pipelineparameter.ipynb) 这两个笔记本。

## <a name="create-a-versioned-pipeline-endpoint"></a>创建版本受控的管道终结点

可以创建包含多个已发布管道的管道终结点。 这样，你在迭代和更新 ML 管道时，就会有一个固定的 REST 终结点。

```python
from azureml.pipeline.core import PipelineEndpoint

published_pipeline = PipelineEndpoint.get(workspace=ws, name="My_Published_Pipeline")
pipeline_endpoint = PipelineEndpoint.publish(workspace=ws, name="PipelineEndpointTest",
                                            pipeline=published_pipeline, description="Test description Notebook")
```

## <a name="submit-a-job-to-a-pipeline-endpoint"></a>将作业提交到管道终结点

可将作业提交到管道终结点的默认版本：

```python
pipeline_endpoint_by_name = PipelineEndpoint.get(workspace=ws, name="PipelineEndpointTest")
run_id = pipeline_endpoint_by_name.submit("PipelineEndpointExperiment")
print(run_id)
```

还可将作业提交到特定的版本：

```python
run_id = pipeline_endpoint_by_name.submit("PipelineEndpointExperiment", pipeline_version="0")
print(run_id)
```

可以使用 REST API 来完成相同的操作：

```python
rest_endpoint = pipeline_endpoint_by_name.endpoint
response = requests.post(rest_endpoint, 
                         headers=aad_token, 
                         json={"ExperimentName": "PipelineEndpointExperiment",
                               "RunSource": "API",
                               "ParameterAssignments": {"1": "united", "2":"city"}})
```

## <a name="use-published-pipelines-in-the-studio"></a>在工作室中使用已发布的管道

也可以从工作室运行已发布的管道：

1. 登录到 [Azure 机器学习工作室](https://studio.ml.azure.cn)。

1. [查看工作区](how-to-manage-workspace.md#view)。

1. 在左侧选择“终结点”。

1. 在顶部选择“管道终结点”。
 ![机器学习的已发布管道列表](./media/how-to-create-your-first-pipeline/pipeline-endpoints.png)

1. 选择要运行的特定管道，使用或查看管道终结点的先前运行的结果。

## <a name="disable-a-published-pipeline"></a>禁用已发布的管道

若要在已发布管道的列表中隐藏某个管道，请在工作室或 SDK 中禁用它：

```python
# Get the pipeline by using its ID from Azure Machine Learning studio
p = PublishedPipeline.get(ws, id="068f4885-7088-424b-8ce2-eeb9ba5381a6")
p.disable()
```

可以使用 `p.enable()` 再次启用它。 有关详细信息，请参阅 [PublishedPipeline 类](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.publishedpipeline?view=azure-ml-py&preserve-view=true)参考。

## <a name="next-steps"></a>后续步骤

- 使用 [GitHub 上的这些 Jupyter Notebook](https://aka.ms/aml-pipeline-readme) 以进一步探索机器学习管道。
- 参阅有关 [azureml-pipelines-core](https://docs.microsoft.com/python/api/azureml-pipeline-core/?view=azure-ml-py&preserve-view=true) 包和 [azureml-pipelines-steps](https://docs.microsoft.com/python/api/azureml-pipeline-steps/?view=azure-ml-py&preserve-view=true) 包的 SDK 参考帮助信息。
- 参阅[操作指南](how-to-debug-pipelines.md)，获取有关调试管道和排查管道问题的提示。
