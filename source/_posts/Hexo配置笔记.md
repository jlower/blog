---
title: Hexo配置笔记
date: 2024-01-05 04:00:00
categories: 笔记
---

## Hexo跳转自己的文章

```md
{% post_link "文章文件名（不要后缀）" "文章标题（可选）" %}
```

> **字段中间如果有空格要加上   " "   （默认都加上   " "   就行了）**

假如同一目录下文章文件名为Hello-World.md：

```md
{% post_link "Hello-World" %}
{% post_link "Hello-World" "你好世界" %}
```

## Hexo中插入PDF

### 安装hexo-pdf

```bash
npm install --save hexo-pdf
pnpm install --save hexo-pdf
```

### 本地PDF引用

#### 引用本地文件夹中的pdf文件

```md
{% pdf "/文件夹名称/引用文档名字.pdf" %}
```

> **字段中间如果有空格要加上   " "   （默认都加上   " "   就行了）**  
> 在source文件夹下创建一个叫pdf的文件夹，把xxx.pdf文件放在这里，然后在_post文件夹中的xxx.md直接使用

```md
{% pdf "/pdf/xxx.pdf" %}
```

#### 最简单的引用方法：把相应pdf文件存到source文件夹下，引用格式为

```md
{% pdf "/引用文档名字.pdf" %}
```

### 引用网络上的PDF文件

```md
{% pdf "https://bugs.python.org/file47781/Tutorial_EDIT.pdf" %}
```

## 使用 Github Action 构建 Hexo 站点并将其部署到 GitHub Pages 的示例工作流程

```yaml
# 构建 Hexo 站点并将其部署到 GitHub Pages 的示例工作流程

name: Deploy Hexo site to Pages

on:
  # 在针对 `main` 分支的推送上运行。如果您
  # 使用 `master` 分支作为默认分支，请将其更改为 `master`
  push:
    branches: [main]

  # 允许您从 Actions 选项卡手动运行此工作流程
  workflow_dispatch:

# 设置 GITHUB_TOKEN 的权限，以允许部署到 GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# 只允许同时进行一次部署，跳过正在运行和最新队列之间的运行队列
# 但是，不要取消正在进行的运行，因为我们希望允许这些生产部署完成
concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  # 构建工作
  build:
    runs-on: ubuntu-latest
    steps:
      # # Github Action 使用官方的 pandoc 渲染器要此步骤
      # - uses: docker://pandoc/core
      #   with:
      #     args: >-  # allows you to break string into multiple lines
      #       --help
      
      # 下载 pandoc 渲染器到 /opt/hostedtoolcache
      - name: Install Pandoc
        run: |
          # install pandoc
          curl -s -L https://github.com/jgm/pandoc/releases/download/3.2.1/pandoc-3.2.1-linux-amd64.tar.gz | tar xvzf - -C /opt/hostedtoolcache/
      
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # 如果未启用 lastUpdated，则不需要
      
      # 如果使用 pnpm，请取消注释
      - name: Install pnpm
        uses: pnpm/action-setup@v4
        with:
          version: latest

      # - uses: oven-sh/setup-bun@v1 # 如果使用 Bun，请取消注释
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm # 或 pnpm / yarn / npm
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Install dependencies
        run: pnpm install # 或 pnpm install / yarn install / bun install / npm ci
      - name: Build with Hexo
        run: |
          # add pandoc to PATH
          export PATH="$PATH:/opt/hostedtoolcache/pandoc-3.2.1/bin"
          pnpm build # 或 npm run build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

# 新式部署到GitHub上,不用配置ssh密钥

  # 部署工作
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    needs: build
    runs-on: ubuntu-latest
    name: Deploy
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4


# 老式部署用ssh可以部署到服务器上，要配置ssh密钥

# jobs:
#   # 构建工作
#   build:
#     runs-on: ubuntu-latest
#     steps:
#       - name: Checkout
#         uses: actions/checkout@v3
#         with:
#           fetch-depth: 0 # 如果未启用 lastUpdated，则不需要
      
#       # 如果使用 pnpm，请取消注释
#       - name: Install pnpm
#         uses: pnpm/action-setup@v2
#         with:
#           version: latest

#       # - uses: oven-sh/setup-bun@v1 # 如果使用 Bun，请取消注释
#       - name: Setup Node
#         uses: actions/setup-node@v3
#         with:
#           node-version: 18
#           cache: pnpm # 或 pnpm / yarn / npm
#       - name: Setup Pages
#         uses: actions/configure-pages@v3
#       - name: Install dependencies
#         run: pnpm install # 或 pnpm install / yarn install / bun install / npm ci
#       - name: Build with Hexo

#         # pnpm 解决方法: [GitHub Action中 pnpm不可以加 -g ]
#         # 加 ./node_modules/.bin/ 文件路径解决GitHub Action 报的 hexo: command not found 的错误
#         # ./node_modules/.bin/hexo clean
#         # ./node_modules/.bin/hexo generate

#         run: |
#           pnpm install hexo
#           pnpm install hexo-cli
#           ./node_modules/.bin/hexo clean
#           ./node_modules/.bin/hexo generate

#     # 部署工作
#         # 4a. 部署到 GitHub 仓库（可选）
#       - name: Deploy to GitHub Pages
#         uses: peaceiris/actions-gh-pages@v3
#         with:
#           # 将之前在Github账号内录入过公钥的SSH生成的密钥添加到当前仓库的 Secrets 中
#           # 在 GitHub 仓库页面的 Settings > Secrets and variables > Actions > Repository secrets 中，点击 “New repository secret”，
#           # 然后将密钥名称设置为 SSH_PRIVATE_KEY，密钥值设置为之前在Github账号内录入过公钥的SSH生成的密钥。
#           deploy_key: ${{ secrets.SSH_PRIVATE_KEY }}
#           external_repository: jlower/blog
#           publish_branch: gh-pages
#           publish_dir: ./public
#           commit_message: ${{ github.event.head_commit.message }}
#           user_name: 'github-actions[bot]'
#           user_email: 'github-actions[bot]@users.noreply.github.com'
      # # 4b. 部署到服务器（可选）
      # - name: Deploy to Server
      #   uses: easingthemes/ssh-deploy@v3
      #   env:
      #     SSH_PRIVATE_KEY: ${{ secrets.SERVER_SSH_KEY }}
      #     ARGS: "-rltgoDzvO --delete"
      #     EXCLUDE: ".well-known, .user.ini"
      #     SOURCE: public/
      #     REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
      #     REMOTE_PORT: ${{ secrets.REMOTE_PORT }}
      #     REMOTE_USER: ${{ secrets.REMOTE_USER }}
      #     TARGET: ${{ secrets.TARGET }}
      # # 4c. 部署到阿里云OSS（可选）
      # - name: Setup Aliyun OSS
      #   uses: manyuanrong/setup-ossutil@master
      #   with:
      #     endpoint: oss-cn-hongkong.aliyuncs.com
      #     access-key-id: ${{ secrets.OSS_ACCESSKEY_ID }}
      #     access-key-secret: ${{ secrets.OSS_ACCESSKEY_SECRET }}
      # - name: Deploy to  Aliyun OSS
      #   run: ossutil cp -rf ./public oss://xaoxuu-com/
```

## 使用 Github Bot 拉取最新依赖更新

```yaml
version: 2
updates:
- package-ecosystem: npm
  directory: "/"
  schedule:
    interval: daily
  open-pull-requests-limit: 20
```

## Hexo 的 _config.yml 配置注意事项

```yaml
# 如果你希望网站部署在 <你的 GitHub 用户名>.github.io 的子目录中：
# 建立名为 <repository 的名字> 的储存库，这样你的博客网址为 <你的 GitHub 用户名>.github.io/<repository 的名字>，repository 的名字可以任意，例如 blog 或 hexo。
# 编辑你的 _config.yml，将 url: 更改为 <你的 GitHub 用户名>.github.io/<repository 的名字>。
# 在储存库中前往 Settings > Pages > Source，并将 Source 改为 GitHub Actions。
# Commit 并 push 到默认分支上。
# 部署完成后，前往 https://<你的 GitHub 用户名>.github.io/<repository 的名字> 查看网站。
url: https://blog.lowoneko.eu.org # http://jlower.github.io/blog
```

> Hexo框架默认会使用 ```hexo-renderer-marked``` 来作为markdown的渲染器，但此渲染器不支持markdown的标准数学公式书写形式，即用 ```$$ $$``` 来包裹LaTeX公式，但此渲染器不支持。此渲染器用 ```hexo-math``` 插件来渲染数学公式，但此插件需要用hexo自定义的麻烦标签。  
> 解决方法：使用官方维护的 ```hexo-filter-mathjax``` 公式插件和其推荐的 ```hexo-renderer-pandoc``` 渲染引擎。  

1. 先移除原先的默认渲染器 ```pnpm uninstall hexo-renderer-marked``` 再移除之前下载的数学公式插件，例如 ```pnpm uninstall hexo-math```
1. 在你的blog根目录下，安装新的渲染器Pandoc依赖 ```pnpm install hexo-renderer-pandoc --save```
1. 安装Pandoc环境，[Pandoc官网安装页面](https://github.com/jgm/pandoc)。如果是使用 GitHub Actions 部署，可以参考上边写的 GitHub Actions 部署配置(要在 GitHub Actions 的环境中下载安装Pandoc并配置全局环境变量)。

> blog根目录下打开 ```_config.yml``` 文件，在文件中加入```hexo-filter-mathjax``` 公式插件的配置

```yml
mathjax:
  tags: none # or 'ams' or 'all'
  single_dollars: true # enable single dollar signs as in-line math delimiters
  cjk_width: 0.9 # relative CJK char width
  normal_width: 0.6 # relative normal (monospace) width
  append_css: true # add CSS to pages rendered by MathJax
  every_page: false # if true, every page will be rendered by MathJax regardless the `mathjax` setting in Front-matter
  packages: # extra packages to load
  extension_options: {}
    # you can put your extension options here
    # see http://docs.mathjax.org/en/latest/options/input/tex.html#tex-extension-options for more detail
```

> 插件 ```markdown-it-emoji``` 支持 hexo-renderer-markdown-it 渲染器解析表情  
> 我要安装插件 ```hexo-filter-github-emojis``` 支持 Pandoc 渲染器解析表情  
> 先 ```pnpm install hexo-filter-github-emojis --save```  
> 然后在 ```_config.yml``` 文件中加入 ```hexo-filter-github-emojis``` 插件的配置

```yaml
githubEmojis:
  enable: true
  className: github-emoji
  inject: true
  styles:
    display: inline
    vertical-align: middle # Freemind适用
  customEmojis:
```

> 将文章链接从中文优化为短数字字母编码  
> 使用 ```hexo-abbrlink``` 插件实现，先 ```pnpm install hexo-abbrlink --save```  
> 然后在 ```_config.yml``` 文件中加入 ```hexo-abbrlink``` 插件的配置  
> 会自动为每个文章加上abbrlink，但我用GitHub Action部署，自动加的abbrlink不会上传到GitHub上，所以每次部署都会跟据当前的文章标题重新生成abbrlink，改了文章标题的话链接也会变，除非手动在文章上输入abbrlink

```yml
# 现在我的设置是要求在同一天内不能有同样标题的文章，否则生成的链接会重复，和默认设置 permalink: ':year/:month/:day/:title/' 一样 (这样要求每篇文章必须要填 date: 创建日期)
# permalink: ':year/:month/:day/:title/' # 这是默认的
permalink: posts/:year:month:day-:abbrlink/ # 设置了 :year:month:day- 所以改文章的创建日期链接会变(文章 date: 后面的是创建日期， update: 后面的是更新日期，不填 update: 则更新时间默认为最新一次网站的部署时间)
# abbrlink config
# 会自动为每篇文章生成 abbrlink (根据文章的标题)，也可以自己手动输入abbrlink(如果文章已经有abbrlink的话就不会再自动生成了，所以第一次自动生成abbrlink后再改文章标题链接也不会变)
# 但我用的是 GitHub Action 部署，自动生成的 abbrlink 不会传到 GitHub 上(每篇文章都没有abbrlink，所以每次上传部署都会重新生成abbrlink)，所以改文章标题会让重新生成的 abbrlink 变掉，文章的链接也会变
abbrlink:
  alg: crc32      #support crc16(default) and crc32
  rep: hex        #support dec(default) and hex
  drafts: false   #(true)Process draft,(false)Do not process draft. false(default) 
  # Generate categories from directory-tree
  # depth: the max_depth of directory-tree you want to generate, should > 0
  auto_category:
     enable: true  #true(default)
     depth:        #3(default)
     over_write: false 
  auto_title: false #enable auto title, it can auto fill the title by path
  auto_date: false #enable auto date, it can auto fill the date by time today
  force: false #enable force mode,in this mode, the plugin will ignore the cache, and calc the abbrlink for every post even it already had abbrlink. This only updates abbrlink rather than other front variables.
```

> **注意** 目前主题主要使用 butterfly 了，不用 vivia

```yaml
# 会优先查找 ./themes 中有没有xxx同名文件夹(若有里面得放GitHub下载的源码，否则生成页面为空)
# 若无则查找 ./node_moudules 中有没有 hexo-theme-xxx 
# (推荐全用 npm/pnpm 下载，省的GitHub Action每次 都要改，因为有些主题作者自己的.gitignore设置了不上传GitHub)
theme: butterfly # butterfly vivia
# 使用  hexo-theme-vivia  主题要禁用归档页面的分页:  [若不添加此配置归档页最多只能显示 10 篇文章]  修改 _config.yml 填写下列配置:
# archive_generator:
#   per_page: 0
```
