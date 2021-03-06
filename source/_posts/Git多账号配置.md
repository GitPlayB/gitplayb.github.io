---
title: Git多账号配置
date: 2019-09-28 14:26:06
tags:
  - Git
categories:
  - BUG制造者
---

有时候我们需要在一台电脑上配置多个Git账号（如公司一个、自己一个），道理还是一样，只是稍微麻烦一点。在这里记录一下操作过程。

<!-- more -->

## 单账号

### SSH/Git的原理

Git使用HTTPS/SSH协议，如果使用https，则每次提交代码时需要输入用户名和密码；如果用ssh，则每次无需输入验证，由Git服务器进行验证。其中，公钥由Git服务器保存，自己本地保存的是私钥，验证时，服务器将公钥与你本地的私钥进行匹配。SSH是非对称加密的一个典型例子。

#### 步骤

1. **生成一对公/私钥**

   在本地Git客户端输入命令：

	```bash
	ssh-keygen -t rsa -C "邮箱地址" 
	```

​		ssh-keygen命令生成一对公/私钥，-t指定秘钥类型为rsa方式，-C为comment（注释）

2. **将公钥保存到服务器**

   将本地生成的公钥（**.pub**）复制下来，粘贴到Git服务器（如GitHub）**New SSH key**的地方，保存为一个可以区分的名字就好了

3. **测试连接**

   在Git客户端测试是否能连接上Git服务器：

   ```bash
   ssh -T git@github.com
   ```

   若看到这样的内容，代表连接成功：

   ```bash
   $ Hi yourname! You've successfully authenticated, but GitHub does not provide shell access.
   ```



##### 注意：

安装过程中会提示你输入**passphrase**，即口令。正常情况下（为了方便）可以不用输入，直接回车；输入的话就是为了防止别人操作你的电脑，也可以登录你的Git。输入过程中字符是不显示的（类似于web表单页面输入密码），出于安全考虑。

## 为什么

来到公司后，由于公司这边用的是GitLab，于是配置了GitLab的SSH。配置了公钥和私钥后可以免密登录，很方便。但是，想要自己新建项目练习的时候发现，公司给的账号是**LDAP账号**，即**域账号**。由于域账号有权限的限制，所以自己什么也做不了，只能新建snippet（代码片段）。所以，想用GitHub来做项目练习。

之前自己有一个GitHub账号，也在自己的电脑上配置了SSH秘钥，即自己电脑可以和GitHub的服务器连上，进行本地代码库的一些操作。于是想在公司电脑上也配置一下秘钥就行了。

但是问题来了——



## 怎么做

第一次进行SSH配置时的配置文件在目录***$USER/Administrator/.ssh***下，包括**id_rsa（私钥）和id_rsa.pub（公钥）**。我如果再配置一个SSH连接，秘钥怎么命名呢？

很自然地想到，可以命名为：**id_rsa_github和id_rsa_github.pub**。但是Git要怎么识别呢？

于是在网上搜索一番，发现Git是可以配置多个账号的。

### 1. 分别生成公/私钥

生成公/私钥的过程和配置一个SSH连接的过程是一样的，同样的步骤✖2就好啦。

需要把不同的连接的公/私钥的名字区分开，并与*config*里的名字对应，这样Git服务器才知道到哪去找对应的秘钥。

### 2. 新建config

在**.ssh**目录下，新建一个配置文件**config（无后缀）**，用于对不同账号的主机进行配置。假如我们想同时配置一个GitLab的账号，一个GitHub的账号，那么我们需要做的就是：

```properties
Host gitlab.com
    HostName gitlab.com
    User yourname@yourhost.com
    PreferredAuthentications publickey
    IdentityFile $USER/Administrator/.ssh/id_rsa_gitlab

Host github.com
    HostName github.com
    User yourname@yourhost.com
    PreferredAuthentications publickey
    IdentityFile $USER/Administrator/.ssh/id_rsa_github
```

可以看到，我们指定了验证身份时用到的是publickey（存放在Git服务器的公钥），并且匹配它的私钥文件是**id_rsa_name**，只要可以通过**name**来区分不同的账号即可。

**User**是你的账号邮箱，也可以与你注册Git平台的邮箱不一致，只是你每次提交代码时的一个身份标记。

### 3. 将私钥添加到ssh-agent
有时，在本地配置好后还是无法克隆项目（`permission denied`）怎么办？这时候，还需要把自己的私钥添加到**ssh-agent**

1. 在Git中输入`eval $(ssh-agent -s)`首先寻找一个ssh代理；

2. 将自己的私钥告诉代理：`ssh-add ~/.ssh/id_rsa`。表示将主目录下的私钥*id_rsa*加入ssh连接。

之后再打开Git，就不需要输入密码啦！Git的操作（clone等等）也都可以在本地进行了。