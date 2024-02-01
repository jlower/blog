---
title: 用pandas包处理csv、txt数据
date: 2024-01-31 03:00:00
categories: 机器学习
---

# 用 pandas 包导入 csv、txt 数据集

## 递归读入所定目录下(包括他的子目录)中的所有 csv、txt 文件并合并为一个 ```DataFrame```对象

```python
import os
import pandas as pd


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
target_directory = '/path/to/your/directory'

# 获取所有CSV和TXT文件的路径
file_paths = read_files_in_directory(target_directory)

# 合并文件为一个DataFrame
dataframe_total = merge_files_to_dataframe(file_paths)

# 打印结果
# print(dataframe_total)
```

## 将 ```DataFrame``` 对象转换为其他易于处理的对象

### 将 ```DataFrame``` 对象转换为 ```numpy.ndarray``` 对象

```python
# 转换成二维 numpy.ndarray 列表
data = dataframe_total.iloc[:, :].values
```

# 确认GPU

```bash
!nvidia-smi
```

# kaggle 默认说明

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
