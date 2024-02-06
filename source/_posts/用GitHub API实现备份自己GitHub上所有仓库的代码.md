---
title: 用GitHub API实现自动拉取自己 GitHub 上所有仓库的代码并打包成 zip 备份
date: 2024-02-05 03:00:00
categories: 实用脚本与解决方案
---

## 小心滥用GitHub被封,尽量使用GitLab

> 使用 ```git config --global credential.helper store``` 只用第一次验证后便永久记住GitHub账号, 适合Linux上使用HTTPS链接GitHub库, Windows有自己的密码管理器不用他
> 使用 ```git config --global --unset credential.helper``` 移除设置, 删除持久化密码

## 脚本与 AList 联合使用

> *subprocess* 会把列表里的每个部分用 ""/'' 引号 包起来, 所以路径有空格时用 ```subprocess``` 无需特殊处理
> *subprocess* 默认将输出打印到控制台 ; 可以更改方法中 ```stdout=``` 参数控制输出
> *subprocess.run* 是阻塞的，会一直卡着等待程序运行
> *subprocess.Popen* 后台执行，不阻塞

```python
from github import Github
# Authentication is defined via github.Auth
from github import Auth
import os
import shutil
import time
import subprocess

"""
    /etc/crontab 中添加定时任务
    51  1   ? * 2   root    python "/root/py_script/用GitHub API实现备份自己GitHub上所有仓库的代码.py" >/dev/null 2>&1
    每周二 1:51 执行脚本
"""

# git config --global credential.helper store
# git设置为登陆一次后记住 用户名与账户密码 ( 先登陆一次, 再执行这个脚本 )
# 若开启双重验证 则密码填写 个人访问令牌

# 替换为你的 GitHub 个人访问令牌
access_token = "YOUR_ACCESS_TOKEN"
# 替换为你的 GitHub 用户名
user_name = "YOUR_USERNAME"

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

repo_list = []  # 储存 存在哪些 远程git仓库
for repo in repos:
    repo_path = os.path.join(backup_folder, repo.name)
    repo_list.append(repo.name)
    print("正在处理: ", repo.name)
    if os.path.exists(repo_path):
        # 如果本地仓库已存在，执行git pull
        os.chdir(repo_path)
        subprocess.run(["git", "pull"])
    else:
        # 如果本地仓库不存在，设定文件夹路径并进行克隆
        subprocess.run(["git", "clone", repo.clone_url, repo_path])
    time.sleep(0.5)

# 删除 backup_folder文件夹 中不在 repo_list列表 中的 过时文件夹(可能是由于远程仓库改名字导致的)
for dir_name in os.listdir(backup_folder):
    dir_path = os.path.join(backup_folder, dir_name)
    if dir_name not in repo_list:
        if os.path.isdir(dir_path):  # 如果是文件夹
            shutil.rmtree(dir_path)

# 将每个文件夹分别压缩并输出到 zip_output_folder 文件夹 (AList挂载的本地网盘文件夹)
files = os.listdir(backup_folder)  # 获取路径下的子文件 (夹) 列表
zip_list = []  # 储存生成了哪些 zip 压缩包
for file in files:
    file_path = os.path.join(backup_folder, file)
    if os.path.isdir(file_path):  # 如果是文件夹
        zip_output_path = os.path.join(zip_output_folder, file)
        zip_output_path = zip_output_path + ".zip"
        zip_list.append(file + ".zip")
        # zip "-o" 选项为覆盖同名文件
        subprocess.run(["zip", "-r", "-o", zip_output_path, file_path])

# 删除 zip_output_folder文件夹 中不在 zip_list列表 中的 过时文件(可能是由于远程仓库改名字导致的)
for filename in os.listdir(zip_output_folder):
    file_path = os.path.join(zip_output_folder, filename)
    if filename not in zip_list:
        os.remove(file_path)

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
