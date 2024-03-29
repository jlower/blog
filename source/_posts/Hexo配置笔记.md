---
title: Hexo配置笔记
date: 2024-01-05 04:00:00
categories: 笔记
---

## Hexo跳转自己的文章

```md
{% post_link 文章文件名（不要后缀） 文章标题（可选） %}
```

假如同一目录下文章文件名为Hello-World.md：

```md
{% post_link Hello-World %}
{% post_link Hello-World 你好世界 %}
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
{% pdf /文件夹名称/引用文档名字.pdf %}
```

> 在source文件夹下创建一个叫pdf的文件夹，把xxx.pdf文件放在这里，然后在_post文件夹中的xxx.md直接使用

```md
{% pdf /pdf/xxx.pdf %}
```

#### 最简单的引用方法：把相应pdf文件存到source文件夹下，引用格式为

```md
{% pdf /引用文档名字.pdf %}
```

### 引用网络上的PDF文件

```md
{% pdf https://bugs.python.org/file47781/Tutorial_EDIT.pdf %}
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
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # 如果未启用 lastUpdated，则不需要
      
      # 如果使用 pnpm，请取消注释
      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: latest

      # - uses: oven-sh/setup-bun@v1 # 如果使用 Bun，请取消注释
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: pnpm # 或 pnpm / yarn / npm
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Install dependencies
        run: pnpm install # 或 pnpm install / yarn install / bun install / npm ci
      - name: Build with Hexo
        run: |
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
url: http://jlower.github.io/blog
```

```yaml
# 会优先查找 ./themes 中有没有xxx同名文件夹(若有里面得放GitHub下载的源码，否则生成页面为空)
# 若无则查找 ./node_moudules 中有没有 hexo-theme-xxx 
# (推荐全用 npm/pnpm 下载，省的GitHub Action每次 都要改，因为有些主题作者自己的.gitignore设置了不上传GitHub)
theme: vivia # butterfly vivia
# 使用  hexo-theme-vivia  主题要禁用归档页面的分页:  [若不添加此配置归档页最多只能显示 10 篇文章]  修改 _config.yml 填写下列配置:
archive_generator:
  per_page: 0
```
