---
title: 用GitHub API实现自动拉取自己 GitHub 上所有仓库的代码并打包成 zip 备份
date: 2024-02-05 03:00:00
categories: 实用脚本与解决方案
---

## 小心滥用GitHub被封,尽量使用GitLab

## 脚本与 AList 联合使用

```python
from github import Github
# Authentication is defined via github.Auth
from github import Auth
import os
import time
import subprocess

# 替换为你的GitHub个人访问令牌
access_token = "YOU TOKEN HERE"

# using an access token
auth = Auth.Token(access_token)

# 创建一个Public Web Github对象
g = Github(auth=auth)

# 获取当前用户
user = g.get_user()

# 获取用户的所有仓库
repos = user.get_repos()

# 备份每个仓库
backup_folder = "/root/Git Repository backup"
zip_output_folder = "/etc/alist/storage/_Git Repository backup"

os.makedirs(backup_folder, exist_ok=True)
subprocess.run(["rm", "-rf", zip_output_folder])
os.makedirs(zip_output_folder, exist_ok=True)

for repo in repos:
    repo_path = os.path.join(backup_folder, repo.name)
    print("正在处理: ", repo.name)
    if os.path.exists(repo_path):
        # 如果本地仓库已存在，执行git pull
        os.chdir(repo_path)
        subprocess.run(["git", "pull"])
    else:
        # 如果本地仓库不存在，设定文件夹路径并进行克隆
        subprocess.run(["git", "clone", repo.clone_url, repo_path])
    time.sleep(0.5)

# 将每个文件夹分别压缩并输出到AList
files = os.listdir(backup_folder)  # 获取路径下的子文件 (夹) 列表
for file in files:
    file_path = os.path.join(backup_folder, file)
    if os.path.isdir(file_path):  # 如果是文件夹
        zip_output_path = os.path.join(zip_output_folder, file)
        zip_output_path = zip_output_path + ".zip"
        subprocess.run(["zip", "-r", zip_output_path, file_path])

print("所有仓库已备份或更新到本地文件夹！")

# 删除第一层级的空文件夹
# files = os.listdir(backup_folder)  # 获取路径下的子文件 (夹) 列表
# for file in files:
#     file_path = os.path.join(backup_folder, file)
#     if os.path.isdir(file_path):  # 如果是文件夹
#         if not os.listdir(file_path):  # 如果子文件为空
#             os.rmdir(file_path)  # 删除这个空文件夹
#             print('Remove: ', file)
```
