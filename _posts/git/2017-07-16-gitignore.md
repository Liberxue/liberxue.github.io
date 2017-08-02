---
layout: blog
istop: true
title: "Git忽略规则及.gitignore规则正确姿势"
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/38390376.jpg
date:  2017-07-16 23:45:56
category: git
tags:
- github
- git
- gitignore
- Git忽略规则
---

# 实现需求
在git中如果想忽略掉某个文件或者文件夹，不想这个文件或者文件夹提交到版本库中，可以使用修改根目录中 .gitignore 文件的方法（如无，则需自己手工建立此文件）。这个文件每一行保存了一个匹配的规则例如：

# 创建gitignore文件

```
touch .gitignore
```
## 注释Git忽略规则
```
# 此为注释 – 将被 Git 忽略
 
*.a       # 忽略所有 .a 结尾的文件
!lib.a    # 但 lib.a 除外
/-liberxuesite     # 仅仅忽略项目根目录下的 liberxuesite 文件，不包括 subdir/liberxuesite
liberxue/    # 忽略 liberxue文件夹/ 目录下的所有文件以及文件夹本身
doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```
## gitignore忽略规则不生效原因

规则很简单，不做过多解释，但是有时候在项目开发过程中，突然心血来潮想把某些目录或文件加入忽略规则，按照上述方法定义后发现并未生效，原因是.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。那么解决方法就是先把本地缓存删除（改变成未track状态），然后再提交：

```
git rm -r --cached .
git add .
git commit -m 'update .gitignore'

```