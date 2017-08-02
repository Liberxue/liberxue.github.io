---
layout: blog
istop: true
title: "GitHub当作私密的版本控制系统远端版本库私有化哈"
background-image: https://o243f9mnq.qnssl.com/2017/06/116099051.jpg
date:  2017-03-07
category: git
tags:
- github
- git-crypt
- encfs
- gpg
- git-remote-gcryp
- sshfs
---
 
## 目的
 
我打算把所有服务器的配置文件用git管理起来，这样可以记录配置变更状况。 但是有一个问题是，如何多人协作？服务器配置信息非常敏感，如果这个版本库泄漏，整个公司的服务器架构就彻底泄漏了。 这个版本库只能在开发者本地电脑里面解密，远程托管版本库的服务器不应该知道文件里面的内容。

那么解决办法就是：本地git版本库是解密的，在上传过程中内容全部加密，密钥保存在本地，同时密钥可以分享给其他开发者。

### 考虑了几个解决方案：

1. ``git-crypt``：可以加密部分文件，原理是加上了加密的fiter和diff， 但是官方说只适合加密部分文件，而不适合全版本库加密。部分文件加密很容易造成信息泄漏，一定要全版本库加密才适合。

2. 串联``sshfs``和远程服务器加密文件系统``encfs``：首先用``sshfs``加载远端文件系统，然后用``encfs``创建加密文件系统。 我估计无法解决多人同时``push``情况下的竞争条件，并且encfs有安全漏洞，使用``上push/pull``之前需要加载两层文件系统，不是很方便。

3. ``git-remote-gcryp``t用``gpg``进行远端加密。 比较符合我预期的模式，但是用``gpg``不是特别方便协作。但是别的方法走不通，只有这个方法可用。

#### 使用方法


######  安装git-remote-gcrypt和gnupg
```
sudo apt-get install git-remote-gcrypt gnupg
```
###### 创建一个gpg的key，
 需要设置用户名，邮箱，描述等，不要设置过期时间
```
gpg --gen-key
```
###### 记录一下生成的key的ID，

比如2048R/liberxue013里面的liberxue013，2048代表加密轮数，越多越不容易破解
```
gpg --list-keys
```
###### 生成一个测试版本库
```
mkdir test1 && cd test1
git init .
echo "test" > a.txt
git add . && git ci -m "update"
```
##### 创建一个测试project

在你的github上面创建一个project，比如：https://github.com/liberxue

######  配置远端加密版本库
```
git remote add cryptremote gcrypt::git@github.com:liberxue/liberxue.git
```
###### 最好指定用哪个key加密
 这样可以共享这个key给其他人用
```
git config remote.cryptremote.gcrypt-participants "liberxue013"
```
###### push到远端
```
git push cryptremote master
```
* 访问远端版本库，看看文件内容，和commit里面的信息，是不是都是加密的？

## 如何分享给其他人


##### 导出key
```
gpg --export-secret-key -a "share@share.com" > secretkey.asc
```
- 把secretkey.asc分享给其他人，拷贝的时候记得先压缩加密一下再发送，更安全

###### 别人电脑里面导入
```
gpg --import secretkey.asc
```
###### 下载代码
```
git clone gcrypt::git@github.com:liberxue/liberxue.git test2 // test2是git clone 在本地的文
件名
```
###### 也要指定一下用什么key加密
```
git config remote.cryptremote.gcrypt-participants "liberxue013"

```

用这种方法，可以用``git``管理一些私密又需要协作的信息（比如服务器配置）， 也可以把github当作私密的版本控制系统来用（commit的消息还是明文的）。



