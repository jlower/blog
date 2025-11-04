---
title: Gradio可视化与打包
date: 2025-01-01 03:00:00
categories: [可视化]
keywords: [Gradio,可视化]
tag: []
description:
---
## Gradio使用记录

### 启动 Gradio 时自动打开本地浏览器网页

```python
# 启动 Gradio 接口
iface.launch(inbrowser=True) # inbrowser=True 则会自动打开本地浏览器网页
# iface.launch(inbrowser=True, share=True) # share=True 则会分享网页链接
```

### Gradio 默认上传文件的文件夹

- Gradio 默认上传文件的文件夹 ``C:\Users\lovo\AppData\Local\Temp\gradio``
- PyInstaller 打包后exe程序文件临时释放的文件夹(需要ctrl+c结束程序才能自动删除文件夹, 直接关闭cmd窗口的话文件夹可能不会自动删除) ``C:\Users\lovo\AppData\Local\Temp\_MEIxxxxxx``

## Gradio打包

### 使用PyInstaller

#### 新建纯净的打包环境( pyinstaller 默认打包环境内的所有包)

```bash
conda create -n temp
activate temp
pip install -r requirements.txt
```

#### 修改spec文件

- Gradio打包需添加

```python
datas = []
datas += collect_data_files('gradio_client')
datas += collect_data_files('gradio')
datas += collect_data_files('safehttpx')
```

```python
    module_collection_mode={ 'gradio': 'py',},
```

- 修改spec文件

```python
    ['gui.py'], # 需要打包的文件名
```

```python
    icon=['camera.ico'], # 图标名称
```

- spec文件总览

```python
# -*- mode: python ; coding: utf-8 -*-
from PyInstaller.utils.hooks import collect_data_files

datas = []
datas += collect_data_files('gradio_client')
datas += collect_data_files('gradio')
datas += collect_data_files('safehttpx')


a = Analysis(
    ['gui.py'],
    pathex=[],
    binaries=[],
    datas=datas,
    hiddenimports=[],
    hookspath=[],
    hooksconfig={},
    runtime_hooks=[],
    excludes=[],
    noarchive=False,
    optimize=0,
    module_collection_mode={ 'gradio': 'py',},
)
pyz = PYZ(a.pure)

exe = EXE(
    pyz,
    a.scripts,
    a.binaries,
    a.datas,
    [],
    name='gui',
    debug=False,
    bootloader_ignore_signals=False,
    strip=False,
    upx=True,
    upx_exclude=[],
    runtime_tmpdir=None,
    console=True,
    disable_windowed_traceback=False,
    argv_emulation=False,
    target_arch=None,
    codesign_identity=None,
    entitlements_file=None,
    icon=['camera.ico'],
)
```
