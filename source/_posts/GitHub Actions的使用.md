---
title: GitHub Actions的使用
date: 2024-02-05 03:00:00
categories: 实用脚本与解决方案
---

## dependabot.yml 依赖拉取机器人

```yml
version: 2
updates:
- package-ecosystem: npm
  directory: "/"
  schedule:
    interval: daily
  open-pull-requests-limit: 20
```

## 设置GitHub Actions定时任务

```yml
# This workflow will install Python dependencies, run tests and lint with a single version of Python
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-python

name: xxx

on:
  schedule:
    # 设置启动时间，为 UTC 时间, UTC 2点 对应 北京时间早上 10 点
    - cron : '58 01 * * *'
  workflow_dispatch:

permissions:
  contents: read

jobs:
  build:

    runs-on: ubuntu-latest
    env:
      TZ: Asia/Shanghai
    steps:
    - uses: actions/checkout@v4
    - name: Set up Python 3.12
      uses: actions/setup-python@v5
      with:
        python-version: "3.12"
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install requests
        pip install datetime
        if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
    - name: Run Message send
      run: |
        python 脚本/xxx.py
```

## GitHub Actions 自动部署网页到 GitHub Pages

```yml
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
