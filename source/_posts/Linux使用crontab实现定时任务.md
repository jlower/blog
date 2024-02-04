---
title: Linux使用crontab实现定时任务
date: 2024-02-05 03:00:00
categories: 实用脚本与解决方案
---

## Ubuntu使用下面任意一条命令查看crontab定时任务服务是否启动

```bash
systemctl status cron
systemctl cron status
```

## root用户 给全局任意用户加任务 修改/etc/crontab文件即可

```bash
vim /etc/crontab
```

## Linux中 如果文件路径有空格需要用双引号将路径括起来 或 加上转义字符\

```bash
cd "/etc/alist/storage/_Git Repository backup"
cd /etc/alist/storage/_Git\ Repository\ backup
```

## 使用 flock timeout 等命令保证定时任务不重复执行或卡死
