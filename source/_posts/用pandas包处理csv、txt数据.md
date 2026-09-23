---
title: 用pandas包处理csv、txt数据
date: 2024-01-31 03:00:00
categories: [机器学习]
keywords: [机器学习,pandas,处理数据,数据]
tag: []
description:
---
## 用 pandas 包导入 csv、txt 数据集

### 读入单个 csv、txt 文件为 ``DataFrame`` 对象

```python
# 导入单个文件
import pandas as pd
import numpy as np

dataframe_total = pd.read_csv("/path/to/target/file")

# 打印结果
print(dataframe_total)
```

### 递归读入所定目录下(包括他的子目录)中的所有 csv、txt 文件并合并为一个 ``DataFrame`` 对象

```python
# 递归读入所定目录下(包括他的子目录)中的所有 csv、txt 文件并合并为一个 ```DataFrame``` 对象

import os
import pandas as pd
import numpy as np


def read_files_in_directory(directory):
    '''
    获取文件夹(包括他的子目录)中所有文件的文件对象,并组成列表返回
    :param directory: 目标文件夹地址
    :return all_files: 返回获取到的所有文件的文件对象列表
    '''
    all_files = []

    for root, dirs, files in os.walk(directory):
        for file in files:
            if file.endswith('.csv') or file.endswith('.txt'):  # 只操作 csv、txt 文件，修改相应后缀就可以操作不同的文件
                file_path = os.path.join(root, file)
                all_files.append(file_path)

    return all_files


def merge_files_to_dataframe(files):
    '''
    读取文件对象列表中的所有文件,生成每个文件对应的 DataFrame 对象组成列表返回
    :param files: 所要读取的文件对象列表
    :return all_dataframes: 返回读取每个文件分别生成的所有 DataFrame 对象列表
    '''
    all_dataframes = []

    for file in files:
        if file.endswith('.csv'):
            df = pd.read_csv(file)
            # df = pd.read_csv(file, header=None)  # 若没有标题则用这句代码, 不带标题
        elif file.endswith('.txt'):
            df = pd.read_csv(file, delimiter='\t')  # 假设 txt 中用 制表符 分隔数据
            # df = pd.read_csv(file, delimiter='\t', header=None)  # 若没有标题则用这句代码, 不带标题
        all_dataframes.append(df)

    merged_dataframe = pd.concat(all_dataframes, ignore_index=True)
    return merged_dataframe


# 定义目标目录
target_directory = '/path/to/target/directory'

# 获取所有CSV和TXT文件的路径
file_paths = read_files_in_directory(target_directory)

# 合并文件为一个DataFrame
dataframe_total = merge_files_to_dataframe(file_paths)

# 打印结果
print(dataframe_total)
```

### 将 ``DataFrame`` 对象转换为其他易于处理的对象

#### 将 ``DataFrame`` 对象转换为 ``numpy.ndarray`` 对象

```python
# 用 .values 转换成 numpy.ndarray 列表
data = dataframe_total.iloc[:, :].values
```

> ``.reshape()`` 是一个用于调整数组形状的NumPy方法。在这里，它被用于将一维数组转换为二维列向量。
> ``.reshape(-1, 1)`` 将⼀维数据变成只有1列，⾏数不知道多少[ -1 代表根据剩下的维度计算出数组的另外⼀个shape属性值]。
> 具体而言，如果有一个形状为 (n,) 的一维数组，使用 ``.reshape(-1, 1)`` 将会将其转换为形状为 (n, 1) 的二维数组，其中每个元素都是一个单独的行向量。这在某些机器学习算法中是必要的，因为它们通常期望输入数据是一个二维数组。
> 在下面示例中，``data = dataframe_total['value'].values.reshape(-1, 1)`` 就是将 'value' 列的一维数据转换为一个列向量。这样，data 变量将成为一个二维数组，每个行代表一个数据点，每个列代表一个特征。在 K-medoids 聚类算法中，这种形式的输入通常是必需的。

```python
# 提取'value'列作为特征 并重新组成二维 numpy.ndarray 列表
data = dataframe_total['value'].values.reshape(-1, 1)
```

```python
# 提取 'value' 列作为特征
data = dataframe_total['value'].values
# 复制 'value' 列的值，生成 data 的第二列
data = np.hstack([data, data])
```

## 一维数据转换成二维数据，满足 ``scikit-learn`` 等库的输入要求

### 若只有一个值、序号代表时间，你仍然可以使用 ``.reshape(-1, 1)`` 来转换数据

在**时间序列**分析中，这种转换通常用于将时间序列数据重塑为适合机器学习模型的格式。例如，如果你有一个时间序列数据，其中每个时间点**只有一个观测值**，你可以将这个一维数组转换为二维数组，以便在模型中使用。
以下是一个简单的Python代码示例，展示了如何对只有一个值的时间序列数据进行转换：

```python
import numpy as np

# 假设有一个时间序列数据，其中序号代表时间
time_series = np.array([100, 110, 105, 115, 120])

# 使用.reshape(-1, 1)将其转换为二维数组
time_series_reshaped = time_series.reshape(-1, 1)

print(time_series_reshaped)
```

## 确认GPU

```bash
!nvidia-smi
```

## kaggle 默认说明

```python
# This Python 3 environment comes with many helpful analytics libraries installed
# It is defined by the kaggle/python Docker image: https://github.com/kaggle/docker-python
# For example, here's several helpful packages to load

# import numpy as np # linear algebra
# import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)

# Input data files are available in the read-only "../input/" directory
# For example, running this (by clicking run or pressing Shift+Enter) will list all files under the input directory

# import os
# for dirname, _, filenames in os.walk('/kaggle/input'):
#     for filename in filenames:
#         print(os.path.join(dirname, filename))

# You can write up to 20GB to the current directory (/kaggle/working/) that gets preserved as output when you create a version using "Save & Run All" 
# You can also write temporary files to /kaggle/temp/, but they won't be saved outside of the current session
```
