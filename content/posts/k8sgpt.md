+++
date = '2025-10-29T09:56:10+08:00'
draft = false
title = 'K8sGPT'
categories= ["devops"]
+++

## 简介

[k8sgpt](https://k8sgpt.ai/) 是一个用于诊断修复 k8s 集群故障的 AI 工具。支持 OpenAI、Azure、Ollama 等 AI 提供商。

## 准备

- AI 提供商。由于政策法规原因，国内无法正常访问国际的 AI 提供商，阿里云的[百炼大模型](https://www.aliyun.com/product/bailian)兼容 OpenAI 接口。接口的具体兼容性，可查看[OpenAI兼容接口](https://help.aliyun.com/zh/model-studio/compatibility-of-openai-with-dashscope?spm=a2c4g.11186623.help-menu-2400256.d_2_10_0.186168edmqzs8m)

- helm

## 安装

k8sgpt 支持命令行与服务端两种部署方式。

### 命令行方式部署

1. 安装命令行工具

    ```shell
    # 使用 curl
    curl -LO https://github.com/k8sgpt-ai/k8sgpt/releases/latest/download/k8sgpt_Linux_x86_64.tar.gz
    tar -xzf k8sgpt_Linux_x86_64.tar.gz
    sudo mv k8sgpt /usr/local/bin/
    ```

    ```shell
    # 使用 wget
    wget https://github.com/k8sgpt-ai/k8sgpt/releases/latest/download/k8sgpt_Linux_x86_64.tar.gz
    tar -xzf k8sgpt_Linux_x86_64.tar.gz
    sudo mv k8sgpt /usr/local/bin/
    ```

    更多安装方式，请参考[CLI Installation](https://k8sgpt.ai/docs/getting-started/installation)

2. 配置 kubeconfig 文件

    k8sgpt 默认读取 `~/.kube/config` 路径的配置文件访问 kubernetes 集群，也可通过 `--kubeconfig` 指定配置文件。

3. 配置 AI 提供商

    ```shell
    k8sgpt auth add --baseurl https://dashscope.aliyuncs.com/compatible-mode/v1 -m qwen-plus-latest -p sk-xxxxxxxxxxxxxxxxx
    ```

    查看已添加的认证配置

    ```shell
    $ k8sgpt auth list
    Default: 
    > openai
    Active: 
    > openai
    Unused: 
    > localai
    > ollama
    > azureopenai
    > cohere
    > amazonbedrock
    > amazonsagemaker
    > google
    > noopai
    > huggingface
    > googlevertexai
    > oci
    > customrest
    > ibmwatsonxai
    ```

### operator 式部署

| operator 方式部署不支持开启 rest/http，可在[k8sgpt.go#L249](https://github.com/k8sgpt-ai/k8sgpt-operator/blob/main/pkg/resources/k8sgpt.go#L249) 行内增加 `"--http"` 标识默认开启。

1. 安装 operator

    ```shell
    helm repo add k8sgpt https://charts.k8sgpt.ai/
    helm repo update
    helm install release k8sgpt/k8sgpt-operator -n k8sgpt-operator-system --create-namespace
    ```

2. 创建 LLM 密钥

    ```shell
    OPENAI_TOKEN=sk-xxxxxxxx
    kubectl create secret generic k8sgpt-secret --from-literal=openai-api-key=$OPENAI_TOKEN -n k8sgpt-operator-system
    ```

3. 【可选】存储 kubeconfig 为 kubernetes secret，集群内访问 kubernetes 可跳过此步骤

    ```shell
    kubectl create secret generic sample-kubeconfig --from-file=kubeconfig=~/.kube/sample-kubeconfig -n k8sgpt-operator-system
    ```

4. 部署 K8SGPT

    替换 `version` 的值为[最新版本](https://github.com/k8sgpt-ai/k8sgpt/releases)，编写此片文档的最新版本为 `v0.4.26`

    ```shell
    kubectl apply -f - << EOF
    apiVersion: core.k8sgpt.ai/v1alpha1
    kind: K8sGPT
    metadata:
      name: k8sgpt
      namespace: k8sgpt-operator-system
    spec:
      ai:
        # 混淆提交的敏感数据
        anonymized: false
        # 自动修复
        autoRemediation:
          enabled: false
          resources:
          - ReplicaSet
          - Pod
          - Deployment
          - Service
          - Ingress
          similarityRequirement: ""
        enabled: true
        baseUrl: https://dashscope.aliyuncs.com/compatible-mode/v1
        model: qwen-plus-latest
        backend: openai
        secret:
          name: k8sgpt-secret
          key: openai-api-key
        language: chinese
      # 分析间隔时间
      analysis:
        interval: "24h"
      # 分析指定资源
      filters: ["Node"]
      noCache: false
      # 限定分析的 namespace
      targetNamespace: "default"
      # 指定外部集群连接文件
      kubeconfig:
        name: sample-kubeconfig
        key: kubeconfig
      version: v0.4.26
    EOF
    ```

## 使用

### 命令行

使用 analyze 子命令分析启动异常的服务。analyze 参数说明：

- `-l` AI 分析回答的语言。
- `-e` 解释问题
- `-L` 标签选择器
- `-n` 被分析资源的命令空间

1. 通过标签分析异常的资源

    ```shell
    $ k8sgpt analyze -l Chinese -e -n at-test -L app=at-marriage-service
    AI Provider: openai

    0: Pod at-test/at-marriage-service-76594df7f-dnk74(Deployment/at-marriage-service)
    - Error: the last termination reason is Error container=at-marriage-service pod=at-marriage-service-76594df7f-dnk74
    Error: 容器at-marriage-service在Pod at-marriage-service-76594df7f-dnk74中启动失败，最后终止原因为Error。  
    Solution: 1. kubectl logs at-marriage-service-76594df7f-dnk74 -c at-marriage-service 查看日志；2. 检查资源配置、镜像是否存在；3. 确认依赖服务是否正常；4. 修复后重新部署。
    ```

## 参考

- [Install K8sGPT](https://github.com/k8sgpt-ai/k8sgpt-operator/tree/main?tab=readme-ov-file#installation)
