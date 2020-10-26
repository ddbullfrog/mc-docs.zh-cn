---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: FileStore - Azure Databricks
description: 了解 FileStore，这是 DBFS 中的一个特殊文件夹，可在其中保存文件并使其可供 Web 浏览器访问。
ms.openlocfilehash: e098949cbe4e77e23d2b3fa3527b4fda4cdc53b6
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121952"
---
# <a name="filestore"></a>FileStore

FileStore 是 [Databricks 文件系统 (DBFS)](databricks-file-system.md) 中的一个特殊文件夹，可在其中保存文件并使其可供 Web 浏览器访问。 可使用 FileStore 执行以下操作：

* 保存在调用 `displayHTML` 时可在 HTML 和 JavaScript 中访问的文件，例如图像和库。
* 保存要下载到本地桌面的输出文件。
* 从本地桌面上传 CSV 和其他数据文件以在 Databricks 上处理。

当你使用某些功能时，Azure Databricks 会将文件放在 FileStore 下的以下文件夹中：

* `/FileStore/jars` - 包含你[上传](../libraries/workspace-libraries.md#uploading-libraries)的库。 如果删除此文件夹中的文件，则工作区中引用这些文件的库可能不再工作。
* `/FileStore/tables` - 包含使用 [UI](data.md#import-data) 导入的文件。 如果删除此文件夹中的文件，则可能无法再访问从这些文件创建的表。
* `/FileStore/plots` - 包含在对 Python 或 R 绘图对象（如 `ggplot` 或 `matplotlib` 绘图）调用 `display()` 时在笔记本中创建的图像。 如果删除此文件夹中的文件，则可能必须在引用它们的笔记本中再次生成这些绘图。 有关详细信息，请参阅 [Matplotlib](../notebooks/visualizations/matplotlib.md) 和 [ggplot2](../notebooks/visualizations/ggplot2.md)。
* `/FileStore/import-stage` - 包含导入[笔记本](../notebooks/notebooks-manage.md#import-notebook)或 [Databricks 存档](../notebooks/notebooks-manage.md#databricks-archive)文件时创建的临时文件。 这些临时文件在笔记本导入完成后消失。

## <a name="save-a-file-to-filestore"></a>将文件保存到 FileStore

若要将文件保存到 FileStore，请将其放在 DBFS 的 `/FileStore` 目录中：

```python
dbutils.fs.put("/FileStore/my-stuff/my-file.txt", "Contents of my file")
```

在以下示例中，请将 `<databricks-instance>` 替换为 Azure Databricks 部署的[工作区 URL](../workspace/workspace-details.md#workspace-url)。

可通过 Web 浏览器 (`https://<databricks-instance>/files/<path-to-file>?o=######`) 访问 `/FileStore` 中存储的文件。 例如，可在 `https://<databricks-instance>/files/my-stuff/my-file.txt?o=######` 访问 `/FileStore/my-stuff/my-file.txt` 中存储的文件，其中 `o=` 后面的数字与 URL 中的数字相同。

## <a name="embed-static-images-in-notebooks"></a><a id="embed-static-images-in-notebooks"> </a><a id="static-images"> </a>在笔记本中嵌入静态图像

可使用 `files/` 位置将静态图像嵌入到笔记本中：

```python
displayHTML("<img src ='files/image.jpg/'>")
```

或者使用 Markdown 图像导入语法：

```
%md
![my_test_image](files/image.jpg)
```

可使用 DBFS [Databricks REST API](../dev-tools/api/index.md) 和 [requests](https://requests.readthedocs.io/en/master/) Python HTTP 库上传静态图像。 如下示例中：

* （将 `<databricks-instance>` 替换为 Azure Databricks 部署的[工作区 URL](../workspace/workspace-details.md#workspace-url)）。
* 将 `<token>` 替换为[个人访问令牌](../dev-tools/api/latest/authentication.md#token-management)的值。
* 将 `<image-dir>` 替换为 `FileStore` 中要上传图像文件的位置。

```python
import requests
import json
import os

TOKEN = '<token>'
headers = {'Authorization': 'Bearer %s' % TOKEN}
url = "https://<databricks-instance>/api/2.0"
dbfs_dir = "dbfs:/FileStore/<image-dir>/"

def perform_query(path, headers, data={}):
  session = requests.Session()
  resp = session.request('POST', url + path, data=json.dumps(data), verify=True, headers=headers)
  return resp.json()

def mkdirs(path, headers):
  _data = {}
  _data['path'] = path
  return perform_query('/dbfs/mkdirs', headers=headers, data=_data)

def create(path, overwrite, headers):
  _data = {}
  _data['path'] = path
  _data['overwrite'] = overwrite
  return perform_query('/dbfs/create', headers=headers, data=_data)

def add_block(handle, data, headers):
  _data = {}
  _data['handle'] = handle
  _data['data'] = data
  return perform_query('/dbfs/add-block', headers=headers, data=_data)

def close(handle, headers):
  _data = {}
  _data['handle'] = handle
  return perform_query('/dbfs/close', headers=headers, data=_data)

def put_file(src_path, dbfs_path, overwrite, headers):
  handle = create(dbfs_path, overwrite, headers=headers)['handle']
  print("Putting file: " + dbfs_path)
  with open(src_path, 'rb') as local_file:
    while True:
      contents = local_file.read(2**20)
      if len(contents) == 0:
        break
      add_block(handle, b64encode(contents).decode(), headers=headers)
    close(handle, headers=headers)

mkdirs(path=dbfs_dir, headers=headers)
files = [f for f in os.listdir('.') if os.path.isfile(f)]
for f in files:
  if ".png" in f:
    target_path = dbfs_dir + f
    resp = put_file(src_path=f, dbfs_path=target_path, overwrite=True, headers=headers)
    if resp == None:
      print("Success")
    else:
      print(resp)
```

### <a name="scale-static-images"></a>缩放静态图像

若要缩小已保存到 DBFS 中的图像，请将图像复制到 `/FileStore`，然后使用 `displayHTML` 中的图像参数重设大小:

```python
dbutils.fs.cp('dbfs:/user/experimental/MyImage-1.png','dbfs:/FileStore/images/')
displayHTML('''<img src="files/images/MyImage-1.png" style="width:600px;height:600px;">''')
```

## <a name="use-a-javascript-library"></a>使用 Javascript 库

此笔记本演示如何使用 FileStore 来包含 JavaScript 库。

### <a name="filestore-demo-notebook"></a>FileStore 演示笔记本

[获取笔记本](../_static/notebooks/filestore.html)