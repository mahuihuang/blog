+++
date = '2025-09-23T21:46:29+08:00'
draft = true
title = 'AWS Command Line Interface'
+++

## 为什么出现 AWS CLI ？

AWS 命令行界面是用于管理 AWS 产品的命令行工具，AWS Command Line Interface 简称 AWS CLI，CLI 并非 Client 的缩写。您可配合脚本文件完成一些简单的任务，本文将示例，如何使用 AWS CLI 新增一个用户，并将用户加入用户组。

运维人员多数时间都是在和命令行，云产品打交道。大部分人的系统性编程能力都较弱，甚者不会写一点代码。即便现在 AI 编程如日中天，我也不敢贸然将 AI 编写的工具来操作生产环境的数据，如果已经熟练掌握某门语言，这倒是可以在审阅之后投入使用。学习使用 AWS CLI 的成本极低，只需要了解所需的子命令和参数即可。

## Linux 安装

| 如需安装其他版本的 CLI，可参考[AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)安装

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

运行 AWS CLI 仅需安装一个二进制文件就可运行，相比编程一个特定的程序要省事许多，直接跳过了搭建环境与编译程序的环节。

## 配置 AWS CLI

命令行的基本用法 `aws [options] <command> <subcommand> [<subcommand> ...] [parameters]`

1. 新增一个 AWS IAM Manage 编程用户，并授予 IAMFullAccess 的访问策略。

2. 执行配置命令

```bash
aws configure
          AWS Access Key ID [None]: accesskey
          AWS Secret Access Key [None]: secretkey
          Default region name [None]: us-west-2
```

## 管理 IAM Center 资源

Identity and Access Management (IAM) 与 IAM Identity Center 差别在于，IAM Identity Center 支持通过 SSO 登录，作者本人当前使用的是 IAM Identity Center，方便与内部的 Authentik 系统集成。

### 获取 IAM Center  信息

若实例为空，则需开通 IAM Center

```bash
aws sso-admin list-instances

{
    "Instances": [
        {
            "InstanceArn": "arn:aws:sso:::instance/ssoins-11111111111",
            "IdentityStoreId": "d-xxxx",
            "OwnerAccountId": "xxxxxxx",
            "CreatedDate": "2025-07-31T13:40:47.568000+08:00",
            "Status": "ACTIVE"
        }
    ]
}
```

### 创建用户组

将 identity-store-id 的值替换为上个章节中获取到的值

```bash
aws identitystore create-group \
    --identity-store-id <d-xxxx> \
    --display-name <Display Name>
```

记录执行名称成功后，返回的 GroupId

### 创建用户

替换下方尖括号内容为实际值

```bash
aws identitystore create-user \
    --identity-store-id <d-xxxx> \
    --user-name <user-name> \
    --display-name <display-name> \
    --emails Value=<email>,Type=Work \
    --region us-east-1 \
    --name  FamilyName=<FamilyName>,GivenName=<GivenName>
```

记录执行名称成功后，返回的 UserID

### 加入用户至用户组

```bash
aws identitystore create-group-membership \
    --identity-store-id <d-xxxx> \
    --group-id <GroupID> \ 
    --member-id UserId=<UserID>
```
