+++
date = '2025-09-24T11:53:36+08:00'
draft = false
title = '如何使用 Vercel 托管 Hugo 项目'
categories= ["devops"]
+++

## 简介

### Hugo

Hugo 是一款静态网站生成框架，官方首页标语称之为世界上最快的构建框架（Hugo The world’s fastest framework for building websites）。它可将 Markdown 渲染为 HTML，并且有多种的主题可用，对 IT 工作者及其利好。

### Vercel

Vercel 与 Github Pages, Cloudflare pages 类似，支持免费计划托管静态网站，集成 Github 完成自动构建。

## 准备

### 安装 hugo

1. Fedora 安装 Hugo

    ```bash
    sudo dnf install hugo
    ```

2. MacOS 安装 Hugo

    ```bash
    brew install hugo
    ```

更多的安装方式请参考[官方文档](https://gohugo.io/installation/)

### 其他

- 安装 [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- 注册 [Vercel](https://vercel.com/) 账号
- 注册 [Github](https://github.com/) 账号

## 创建与部署

### 创建一个 Hugo

查询已安装的 Hugo 版本

```bash
hugo version
```

1. 创建一个 hugo 项目

    ```bash
    hugo new site quickstart
    cd quickstart
    ```

2. 初始化项目为 git 仓库

    ```bash
    git init
    ```

3. 克隆一个主题为 git 子模块，获取更多的[主题](https://themes.gohugo.io/)

    ```bash
    git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
    ```

4. 配置主题，hugo 默认从 theme 目录读取主题

    ```bash
    echo "theme = 'ananke'" >> hugo.toml
    ```

5. 启动项目，仅渲染不生成静态文件

    ```bash
    hugo server
    ```

### 创建 Github 仓库

1. 创建一个 Github 仓库

2. 将本地的 git 仓库推送至 github 仓库

### 部署项目

1. 关联 GitHub 账号，并授权新增的 hugo 项目仓库

2. 新增 vercel 项目，选择已 Github 中已授权的 hugo 项目.
    - 选择 hugo 构建框架（Framework Preset）
    - 指定构建 hugo 的版本，在 *Environment Variables* 中新增 HUGO_VERSION 变量，填写数字版本号。例如：0.150.0，默认版本较低，可能会出现构建失败，或渲染不符合预期的情况

3. 【可选】调整构建命令，在原有的构建命令后追加 `-b https://<domain>/` 参数，domain 为 vercel 自动分配的域名，可在 Settings > Domains 中查看。完成的命令为 `hugo --gc -b https://domain/`。这一步骤的目的是在编译时动态修改站点域名，也可在 `hugo.toml` 中指定 baseURL 值。
