---
title: "Git 配置及SSH key"
description: "Git 配置及SSH key过程记录"
date: 2017-07-01 12:00:00
draft: false
tags: ["Git"]
categories: ["Git"]
---

​	本地 Git 仓库和 GitHub 仓库之间的传输是通过 SSH 加密的，所以配置SSH key之后，上传代码到`Github`远程仓库时就不用输入密码了。一般是在C盘用户目录下有一个 `something` 和 `something.pub` 来命名的一对文件，这个 `something` 通常就是 `id_dsa` 或 `id_rsa`。有 `.pub` 后缀的文件就是公钥，另一个文件则是密钥。连接时必须提供一个公钥用于授权，没有的话就要生成一个。

<!--more-->

> 更多文章欢迎访问我的个人博客-->[幻境云图](https://www.lixueduan.com/)

## 1. Git 配置

配置全局用户名和密码，git提交代码时用来显示你身份和联系方式，并不是github用户名和邮箱

```java
git config --global user.name "lillusory" //改成自己的
git config --global user.email "xueduanli@163.com"  //改成自己的
```

## 2. 生成SSH key

### 2.1 生成秘钥

- 执行`ssh-keygen -t rsa -C "你的邮箱地址" ` 命令 生成ssh key
- 然后会叫你输入保存路径，直接按回车即可，保存在C盘用户目录下
- 然后会提示输入密码和确认密码，不用输入直接按两下回车即可

到这里SSH key就生成好了，接下来就是配置到github上。

### 2.2 配置SSH key

![](https://github.com/illusorycloud/illusorycloud.github.io/raw/hexo/myImages/git/2018-12-27-git-ssh-key-set1.png)

![](https://github.com/illusorycloud/illusorycloud.github.io/raw/hexo/myImages/git/2018-12-27-git-ssh-key-set2.png)



登陆Github-->点击头像-->Settings-->SSH and GPG keys-->选择SSh keys上的New SSH keys-->name 随便写，key就是刚才生成的文件中的所有内容。

文件默认是在C盘用户目录下，我的是`C:\Users\13452\.ssh`

文件夹中应该会有两个文件 ：`id_rsa`和`id_rsa.pub` 

`id_rsa.pub`就是我们要的key, 一般以`ssh-rsa`开头，以你刚才输的邮箱结尾。

### 2.3 测试

执行`ssh -T git@github.com`命令验证一下。

可能会提示，`无法验证主机的真实性`是否要建立连接，输入`yes`就行了。

如果，看到：

> Hi xxx! You've successfully authenticated, but GitHub does not # provide shell access.

恭喜你，你的设置已经成功了。

## 3. 参考

[Git Book](https://git-scm.com/book/zh/v2/%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84-Git-%E7%94%9F%E6%88%90-SSH-%E5%85%AC%E9%92%A5)