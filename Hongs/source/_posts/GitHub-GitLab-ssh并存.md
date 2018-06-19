---
title: GitHub & GitLab ssh并存
date: 2018-05-24 14:09:10
toc: true
categories: 工具
tags: [ssh,github,gitlab]
---
解决GitHub、GitLab的SSH冲突问题
<!-- more -->
## 场景
一台PC上同时使用GitHub、GitLab提交代码，遇到SSH冲突问题。

## 解决方法
分别生成各自SSH，并通过config配置使用场景。
### 1. 生成SSH

```
# github's ssh
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa_github -C "yourGitHubEmail@example.com"
# gitlab's ssh
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa_gitlab -C "yourGitLab@example.com"
```
### 2. 添加private key


```
$ ssh-add ~/.ssh/id_rsa_github
$ ssh-add ~/.ssh/id_rsa_gitlab
```
### 3. 上传public key
在github、gitlab中分别输入对应的~/.ssh/id_isa_gitXXX.pub内容
### 4. 修改配置
打开config

```
open ~/.ssh/config
```
增加以下内容

```
#gitlab
Host gitlab
        HostName gitlab.*.com
        IdentityFile ~/.ssh/id_rsa_gitlab

#github
Host github
        HostName github.com
        IdentityFile ~/.ssh/id_rsa_github
```

### 5. 验证

```
# 测试github
$ ssh -T yourGitHubEmail@example.com
 
# 测试gitlab
$ ssh -T yourGitLab@example.com
```