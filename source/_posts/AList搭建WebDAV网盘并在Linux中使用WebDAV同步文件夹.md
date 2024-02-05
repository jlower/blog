---
title: AList搭建WebDAV网盘并在Linux中使用WebDAV同步文件夹
date: 2024-02-05 03:00:00
categories: 实用脚本与解决方案
---

## AList 搭建 WebDAV 网盘【AList网页端操作方便，对国内网盘支持较好】

> **AList太好用了!!!** 同时管理多个网盘, 批量备份设置文件到多个网盘

## Rclone "rsync for cloud storage"【Rclone也支持不少网盘，方便挂载硬盘与增量同步文件夹, 功能强大 ; 但对国内的网盘支持较少，命令行操作适合Linux, 可以配合AList使用】

> Rclone与AList配合WebDAV同步 , 实现多网盘同步文件夹
> 程序崩溃后可能会在 /root 文件夹下生成 core 文件, **占用空间很大** 直接删除

- Linux上执行 [rclone 的下载脚本](https://rclone.org/install/) ```curl https://rclone.org/install.sh | sudo bash```
- Linux上执行命令时路径如果 **有空格** 要用 ""/'' 引号 包起来
- rclone 可以使用 ```rclone config``` 交互式配置 ; ```rclone config file``` 可以查看配置文件存放的位置, 默认配置完成的后的配置信息保存在 ```/root/.config/rclone/rclone.conf``` 中
- [rclone sync](https://rclone.org/commands/rclone_sync/) 单向同步(安全,不会更改source:path中的文件)

```bash
rclone sync source:path dest:path [flags]
--create-empty-src-dirs   Create empty source dirs on destination after sync
--check-first                                 Do all the checks before starting transfers
-P/--progress flag to view real-time transfer statistics
-n, --dry-run         Do a trial run with no permanent changes

rclone sync /etc/alist/storage/test 迅雷云盘:/test --create-empty-src-dirs --check-first --progress
```

- [rclone bisync](https://rclone.org/commands/rclone_bisync/) 双向同步(危险操作,小心)

### Python执行脚本

> ```subprocess``` 会把列表里的每个部分用 ""/'' 引号 包起来, 所以路径有空格时用 ```subprocess``` 无需特殊处理
> ```subprocess``` 默认将输出打印到控制台 ; 可以更改方法中 ```stdout=``` 参数控制输出
> ```subprocess.run``` 是阻塞的，会一直卡着等待程序运行
> ```subprocess.Popen``` 后台执行，不阻塞

```python
import subprocess

"""
    
    /etc/crontab 中添加定时任务
    53  3   * * *   root    python "/root/py_script/Rclone与AList配合WebDAV同步.py" >/dev/null 2>&1
    每天 3:53 执行脚本
    
    测试选项 开启参数"-n, --dry-run" Do a trial run with no permanent changes
    
    只更改 OneDrive 上的文件, 每天自动同步到各个网盘一次
    - "_Git Repository backup" 文件夹由 VPS 每星期拉取 GitHub 仓库并备份
    - 其他文件夹 ["_WebDAV", "_常用设置留档", "应用"] 是从 OneDrive 上同步
    
    ["OneDrive", "谷歌云盘", "Yandex网盘"] 不同步 "LAPTOP-" 文件夹 只同步 ["_Git Repository backup", "_WebDAV", "_常用设置留档", "应用"] 文件夹
    ["阿里云盘Open", "中国移动云盘", "百度网盘", "迅雷云盘", "天翼云盘客户端"] 同步 ["LAPTOP-", "_Git Repository backup", "_WebDAV", "_常用设置留档", "应用"] 文件夹
    
    执行流程
    - 先把 VPS 上 手动 备份的 ["_Git Repository backup"] 文件夹 从本地 根目录'/' 同步到 OneDrive 根目录'/'下
    - 再从 OneDrive 根目录'/' 同步 ["_WebDAV", "_常用设置留档", "应用"] 文件夹 到 本地 根目录'/'下
    - 再从 本地 根目录'/' 同步 ["_Git Repository backup", "_WebDAV", "_常用设置留档", "应用"] 文件夹 到 ["谷歌云盘", "Yandex网盘", "阿里云盘Open", "中国移动云盘", "天翼云盘客户端", "百度网盘", "迅雷云盘"] 的 "/_同步/" 目录下
    
"""

# 要先下载 rclone 详见 https://rclone.org/install/
# 先执行 rclone config 交互式配置好远端 再运行此脚本

# 本地文件夹路径( 路径最后不要加斜杠'/' )
local_folder = "/etc/alist/storage"

# WebDAV 云端远程名称（根据你的配置设置）
remote_name = "OneDrive"
print("===================== Now processed remote: ", remote_name, " =====================")
# 使用 rclone 从 本地 根目录'/' 同步 ["_Git Repository backup"] 文件夹 到 OneDrive 根目录'/'下
backup_floders = ["_Git Repository backup"]
for backup_floder in backup_floders:
    print("Now processed floder: ", backup_floder)
    subprocess.run(["rclone", "sync", local_folder + f"/{backup_floder}", f"{remote_name}:/{backup_floder}",
                    "--create-empty-src-dirs",
                    "--check-first", "--progress"])
# 使用 rclone 从 OneDrive 根目录'/' 同步 ["_WebDAV", "_常用设置留档", "应用"] 文件夹 到 本地 根目录'/'下
backup_floders = ["_WebDAV", "_常用设置留档", "应用"]
for backup_floder in backup_floders:
    print("Now processed floder: ", backup_floder)
    subprocess.run(["rclone", "sync", f"{remote_name}:/{backup_floder}", local_folder + f"/{backup_floder}",
                    "--create-empty-src-dirs",
                    "--check-first", "--progress"])

# WebDAV 云端远程名称（根据你的配置设置）
remote_names = ["谷歌云盘", "Yandex网盘", "阿里云盘Open", "中国移动云盘", "天翼云盘客户端", "百度网盘", "迅雷云盘"]
# 使用 rclone 从 本地 根目录'/' 同步 ["_Git Repository backup", "_WebDAV", "_常用设置留档", "应用"] 文件夹 到 remote_names 的 "/_同步/" 目录下
backup_floders = ["_Git Repository backup", "_WebDAV", "_常用设置留档", "应用"]
for remote_name in remote_names:
    print("===================== Now processed remote: ", remote_name, " =====================")
    for backup_floder in backup_floders:
        print("Now processed floder: ", backup_floder)
        subprocess.run(["rclone", "sync", local_folder + f"/{backup_floder}", f"{remote_name}:/_同步/{backup_floder}",
                        "--create-empty-src-dirs",
                        "--check-first", "--progress"])

print("ALL SYNC DONE!")
```

## WebDAV使用 rclone/cadaver、rsync 来同步文件夹 ; 批量备份设置文件到 AList 的多个网盘

### rsync 增量同步文件夹【Linux本地命令快速增量同步文件夹】

### cadaver【WebDAV轻量级客户端，Kali自带】

> 连接方法 ```cadaver http://localhost:5244/dav``` 链接AList的dav服务
> cadaver里面的大部分操作和ftp命令行客户端很像
> 输入help可以查看有什么命令, 输入 help [命令] 可以查看具体说明
> 输入bye可以退出
