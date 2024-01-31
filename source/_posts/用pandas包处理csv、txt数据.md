---
title: 用pandas包处理csv、txt数据
date: 2024-01-31 03:00:00
categories: 机器学习
---

# 用 pandas 包处理 csv、txt 数据

## 用 pandas 包导入 csv、txt 数据集

```python
import os
import pandas as pd


def each_file(filedir, data_list):
 '''
    递归读取每个文件夹，将 csv txt 文件的数据读取并保存到 data_list 中
    :param filepath: 想要获取的文件的目录
    :param data_list: 读取每个csv文件保存的pandas的DataFrame列表
    :return: null
    '''
 l_dir = os.listdir(filedir)  # 读取目录下的文件或文件夹
 for one_dir in l_dir:  # 进行循环
  full_path = os.path.join('%s/%s' % (filedir, one_dir))  # 构造路径
  if os.path.isfile(full_path):  # 如果是文件类型就执行操作
   # 只操作 csv txt 文件，修改相应后缀就可以操作不同的文件
   if (one_dir.split('.')[-1] == 'csv') or (one_dir.split('.')[-1] == 'txt'):
    # 这个是操作的语句，最关键的
    # 读取csv文件并保存到列表
    df_tmp = pd.read_csv(full_path, header=None)  # 不带标题
    # df_tmp = pd.read_csv(full_path)  # 每个文件带标题
    data_list.append(df_tmp)
  else:  # 不为文件类型就继续递归
   each_file(full_path, data_list)  # 如果是文件夹类型就有可能下面还有文件，要继续递归


# 获取当前脚本文件所在文件夹夹的路径
# filedir = os.getcwd()

# 指定要遍历的目录路径
filedir = "/kaggle/input/ncep-test-1"
data_list = []
each_file(filedir, data_list)
df_total = pd.concat(data_list, ignore_index=False)
# print(df_total)

# 转换成二维numpy.ndarray列表
data = df_total.iloc[:, :].values
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
