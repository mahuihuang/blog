+++
date = '2025-09-22T21:38:51+08:00'
draft = false
title = 'Jumpserver'
+++
# Jumpserver

[Jumpserver](https://www.jumpserver.org/) 是由[飞致云](https://www.fit2cloud.com/)开源的堡垒机管理工具。作为运维人员，登录 Linux 主机是非常高频的事项。如果登录 Jumpserver 开启了 MFA，重复输入动态验证码是令人非常痛苦的环节。本文介绍如何一键登录 Jumpserver，无需手动输入密码与验证码。

# 配置 SSH 密钥

1. 开启 Jumpserver ssh 客户端端口，Jumpserver 的[默认 ssh 端口](https://docs.jumpserver.org/zh/v3/installation/network_port/)为 2222

2. 上传 ssh 密钥
    登录 Jumpserver 的网页端，并切换至“个人信息”界面，将个人主机的 ssh 公钥更新至认证配置。

3. 验证 ssh 免密登录，进入输入验证码界面表示 ssh 密钥配置成功
    ```bash
    ssh username@example.com -p 2222
    ```

# 配置 totp-cli

## 安装 top-cli

方式一，二进制安装

```bash
curl -Ls https://github.com/yitsushi/totp-cli/releases/latest/download/totp-cli_Linux_x86_64.tar.gz | sudo tar -xzf - -C /usr/local/bin
```

方式二，源码安装

```bash
go install github.com/yitsushi/totp-cli@latest
```

## 导入 totp 密钥

提前准备一个用于获取 totp 随机密钥的密码，并妥善保管。

```bash
export namespace=jumpserver
export account=<username>
/usr/local/bin/totp-cli add-token $namespace $account

Namespace: jumpserver
Account: <username>
Token: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Password: <输入密码>
```

## 验证生成的设备验证码

执行命令后输入生成验证码所需的密码

```bash
totp-cli generate $namespace $account
```

# 配置一键登录脚本

1. 安装 expect

```bash
# Debian
apt install expect

#  RHEL
dnf install expect
```

2. 编写 shell 脚本

```bash
touch /usr/local/bin/jumpserver && sudo chmod +x /usr/local/bin/jumpserver
```

脚本内容
```bash
#!/bin/bash

# 设置上一个章节写入的 totp-cli 密码
export TOTP_PASS=<password>
export namespace=jumpserver
export account=<username>

# 生成 TOTP 验证码
TOTP_CODE=$($HOME/go/bin/totp-cli generate $namespace $account)
if [ -z "$TOTP_CODE" ]; then
  echo "Error: Failed to generate TOTP code"
  exit 1
fi
echo "Generated TOTP Code: $TOTP_CODE"

# 确保 TERM 环境变量
export TERM=xterm

# 使用 expect 自动化 SSH 登录,替换 username 和 example.com 为实际的值
expect -c 'spawn ssh username@example.com -p 2222 -o StrictHostKeyChecking=no; expect ":";send '"$TOTP_CODE\r"'; interact'
```

3. 验证

```bash
/usr/local/bin/jumpserver

Generated TOTP Code: 128627
spawn ssh username@example.com -p 2222 -o StrictHostKeyChecking=no
username
Please Enter MFA Code.
(username@example.com) [OTP Code]: 128627

		xxx,  JumpServer 开源堡垒机

	1) 输入 部分IP，主机名，备注 进行搜索登录(如果唯一).
	2) 输入 / + IP，主机名，备注 进行搜索，如：/192.168.
	3) 输入 p 进行显示您有权限的资产.
	4) 输入 g 进行显示您有权限的节点.
	5) 输入 h 进行显示您有权限的主机.
	6) 输入 d 进行显示您有权限的数据库.
	7) 输入 k 进行显示您有权限的Kubernetes.
	8) 输入 r 进行刷新最新的机器和节点信息.
	9) 输入 s 进行中文-English-日本語语言切换.
	10) 输入 ? 进行显示帮助.
	11) 输入 q 进行退出.
Opt>
```
