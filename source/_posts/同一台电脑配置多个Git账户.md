---
title: 同一台电脑配置多个Git账户
date: 2019-10-212 17:48:48
categories:
- Web前端
tags:
- git
---

工作用的 git 账号和私人用的不同时，可以按此方法配置多个 git 账户。

<!--more-->

## 多账户连接

`$ ssh-keygen -t rsa -C "my_email@email.com"`

输入ssh-key的名字,个人。

`$ ssh-keygen -t rsa -C "company_email@email.com"`

输入ssh-key的名字，公司。

分别将两个rsa.pub公钥添加到主页（个人和公司）。

上面会让你输入公私钥的文件名，默认是Id_rsa， 注意区分

在.ssh文件夹下建立config文件：

```bash
# 配置github.com
Host github.com
    HostName github.com
    IdentityFile /Users/hibiki/.ssh/id_rsa_personal
    PreferredAuthentications publickey
    User github.com

# 配置gitlab
Host git.companyname.com
    HostName git.companyname.com
    IdentityFile /Users/hibiki/.ssh/id_rsa_company
    PreferredAuthentications publickey
    User gitlab
```

Host后面的名字可以随便起，就是命名。

Hostname 是域名 或者ip ,第二个中是我在公司局域网的域名，已经在host文件中配置ip,

注意有个port参数，这个要看你git项目的链接参数才可知道这里要填的端口号，而不是网站服务器的端口号。

User 你的用户名

PreferredAuthentications 验证方式，这里是公钥方式，还可以设置用密码等。

IdentifyFile 是私钥 的文件地址。

## 测试配置是否成功

`ssh -T git@github.com`

这里的`github.com`是上面的Host 的名称。可以分别验证。

## 添加到ssh-agent

`ssh-add 密钥文件路径`

如进到.ssh文件夹中，可以直接添加了 `ssh-add 密钥文件的名字`

如果执行`ssh-add ...`命令提示如下错误：

```
Could not open a connection to your authentication agent.
```

那么请执行`eval $(ssh-agent)`命令后再重试，如果还不行，请再执行`ssh-agent bash`命令后重试。 如果还不行，请参考：[StackOverFlow·ssh-Could not open a…](http://stackoverflow.com/questions/17846529/could-not-open-a-connection-to-your-authentication-agent)

## **配置局部用户和邮箱**

进到项目目录，比如公司的项目和github上个人的项目， 分别配置用户和邮箱。

`git config --local user.name "你的名字"`

`git config --local user.email "你的邮箱"`

配置完可以用以下命令查看当前 git 用户信息，防止配错
`git config user.name`
`git config user.email`

> 修改用户名和邮箱地址 
`git config --global user.name  "xxxx"`
`git config --global user.email  "xxxx"`


可以写一个脚本来自动配置局部用户和邮箱信息：
```bash
#!/bin/bash
echo "更改前的 git 信息为"
echo "user.name:"
git config user.name;
echo "user.email:"
git config user.email;

echo "--------------------------"

git config --local user.name "local_username";
git config --local user.email "local_email@gmail.com";

echo "现在的 git 信息为"
echo "user.name:"
git config user.name;
echo "user.email:"
git config user.email;
```
