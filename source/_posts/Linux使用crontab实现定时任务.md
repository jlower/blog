---
title: Linux使用crontab实现定时任务
date: 2024-02-05 03:00:00
categories: [实用脚本与解决方案]
keywords: [脚本,实用脚本,crontab,Linux,服务器,定时任务]
tag: []
description:
---
## 注意: cron定时任务一般结尾加上 ``>/dev/null 2>&1`` 将输出抛弃不打印到控制台上, 否则会占用大量资源, 卡

> 例如 向/etc/crontab 中添加定时任务 : ***每周二 1:51 执行脚本***
> ``51  1   ? * 2   root    python "/x/xx/xxx.py" >/dev/null 2>&1``

## Linux中 如果文件路径 有空格 需要用双引号将路径括起来 或 加上转义字符\

> *subprocess* 会把列表里的每个部分用 ""/'' 引号 包起来, 所以路径有空格时用 ``subprocess`` 无需特殊处理

```bash
cd "/etc/alist/storage/_Git Repository backup"
cd /etc/alist/storage/_Git\ Repository\ backup
```

## Ubuntu使用下面任意一条命令查看crontab定时任务服务是否启动

```bash
systemctl status cron
systemctl cron status
```

## root用户 给全局任意用户加任务 修改/etc/crontab文件即可

```bash
vim /etc/crontab
```

## 使用 flock timeout 等命令保证定时任务不重复执行或卡死

## 要 永久 运行任务可以使用 ``nohup`` 命令 配合 ``&``后台执行

> ``jobs -l`` 可以查看后台运行的程序
> ``kill -9 [pid]`` 杀死后台运行的 ``nohup`` 程序
