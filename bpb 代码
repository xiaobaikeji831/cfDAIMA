name: Auto Update Worker

on:
  push:
    branches:
      - main
  schedule:
    - cron: "0 1 * * *" # 每天凌晨1点自动运行
  workflow_dispatch:     # 支持手动运行

permissions:
  contents: write

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - name: 初始化仓库
        uses: actions/checkout@v4

      - name: 获取当前本地版本
        id: get_local_version
        run: |
          echo -e "\033[34m[获取本地版本]\033[0m"
          if [ -f version.txt ]; then
            LOCAL_VERSION=$(cat version.txt)
            echo "当前本地版本: $LOCAL_VERSION"
          else
            echo "首次同步，没有本地版本。"
            LOCAL_VERSION=""
          fi
          echo "LOCAL_VERSION=$LOCAL_VERSION" >> $GITHUB_ENV

      - name: 获取最新 Release 信息
        id: get_release
        run: |
          echo -e "\033[34m[获取最新 Release]\033[0m"
          API_URL="https://api.github.com/repos/bia-pain-bache/BPB-Worker-Panel/releases"
          RESPONSE=$(curl -s "$API_URL")
          LATEST_RELEASE=$(echo "$RESPONSE" | jq -r '.[0]')
          TAG_NAME=$(echo "$LATEST_RELEASE" | jq -r '.tag_name')
          DOWNLOAD_URL=$(echo "$LATEST_RELEASE" | jq -r '.assets[] | select(.name == "worker.zip") | .browser_download_url')

          if [ -z "$DOWNLOAD_URL" ] || [ "$DOWNLOAD_URL" == "null" ]; then
            echo -e "\033[31m未找到 worker.zip，退出！\033[0m"
            exit 1
          fi

          echo "最新版本号: $TAG_NAME"
          echo "DOWNLOAD_URL=$DOWNLOAD_URL" >> $GITHUB_ENV
          echo "TAG_NAME=$TAG_NAME" >> $GITHUB_ENV

      - name: 判断是否需要更新
        id: check_update
        run: |
          echo -e "\033[34m[判断是否需要更新]\033[0m"
          if [ "$LOCAL_VERSION" = "$TAG_NAME" ]; then
            echo -e "\033[32m已经是最新版本，无需更新。\033[0m"
            echo "UPDATE_NEEDED=false" >> $GITHUB_ENV
          else
            echo -e "\033[33m发现新版本，需要更新！\033[0m"
            echo "UPDATE_NEEDED=true" >> $GITHUB_ENV
          fi

      - name: 如果需要，清理旧文件并下载新版本
        if: env.UPDATE_NEEDED == 'true'
        run: |
          echo -e "\033[34m[清理旧文件]\033[0m"
          rm -rf ./*
          echo -e "\033[34m[下载最新 worker.zip]\033[0m"
          wget -O worker.zip "$DOWNLOAD_URL"
          echo -e "\033[34m[解压 worker.zip]\033[0m"
          unzip worker.zip
          echo -e "\033[34m[删除 worker.zip]\033[0m"
          rm worker.zip
          echo -e "\033[34m[记录新版本号]\033[0m"
          echo "$TAG_NAME" > version.txt

      - name: 提交更改
        if: env.UPDATE_NEEDED == 'true'
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: "🔄 自动同步最新 Worker 版本：${{ env.TAG_NAME }}"
          commit_author: "github-actions[bot] <github-actions[bot]@users.noreply.github.com>"
          push_options: --force
