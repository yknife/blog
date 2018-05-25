---
title: 使用git管理hexo（一）之同步文章篇
tags: 
 - git
comments: true
---
### 场景
我一直以来都是知道hexo支持git，但是碍于对git的不甚了解，以及害怕折腾，一直都是使用ssh传输文件，最近决定熟悉一下git,所以刚刚好试试git来管理hexo。
<!--more-->
### 部署流程图
![部署流程图](https://wx4.sinaimg.cn/mw690/7d1cdd3agy1frnp3mecswj21kw148djg.jpg)

### 优点
1. 只是把文章的md文件上传到github进行存储，具体编译出来的页面没有上传到github
2. hexo服务器目录中有一个source/_posts的目录，hexo用这个目录来存放文章md文件，用本地的文章文件夹和这个目录进行关联就可以达到分布式管理的效果，hexo g编译起文章来也没有任何冲突
3. 与hexo官网的部署方式不同，官网默认的部署方式是将编译出页面包括css等目录都提交到github上，我觉得这样的部署方式并不好，不方便迁移
```
//官网默认部署，_config.xml中的配置，我的方式不需要这个配置
 deploy:
   type: git
```
### 缺点
还是有许多步骤需要手动完成，比如每次写完文章，都要放到本地的git文件夹，然后push，pull，hexo g一顿操作，过程繁琐

### 改进
我最终想达到的目标，在本地编写md文档完成后，hexo服务器就能感知到文章的增加，并生成、部署出全新的页面。

### 方案
使用Travis CI持续集成，详情请参见[使用git管理hexo（二）之持续集成篇]()。

