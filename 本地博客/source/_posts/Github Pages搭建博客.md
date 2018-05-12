---
title: Github Pages搭建博客
date: 2018-05-09 22:37:21
tags:  #Github,Pages,blog
keywords:  #Github,Pages,blog
---
## 一：Github设置
* 1、注册属于你自己的Github账号
* 2、创建仓库
    * 在Github首页右上角头像左侧加号点选择 New repositor(新存储库),如：hongmaju.github.io
    ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/GithubPages搭建博客1.jpg)
* 3、开启Github Pages
    * 点击Choose a theme选择一个主题，然后点select theme
    ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/GithubPages搭建博客2.jpg)
     ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/GithubPages搭建博客3.png)
## 二：Hexo设置
* 1、安装Node.js
* 2、安装Git
* 3、安装Hexo
    * 在你需要安装Hexo的目录下(新建一个文件夹)右键选择 Git Bash
```python
    npm install hexo-cli -g
    hexo init #初始化网站
    npm install
    hexo g #生成或 hexo generate
    hexo s #启动本地服务器 或者 hexo  server
```
这一步之后就可以通过http://localhost:4000  查看了

## 三：Hexo部署
* 安装部署插件
npm install hexo-deployer-git --save
* 修改根目录下_config.yml文件
```python
deploy:
    type: git
    repo: http://github.com/hongmaju/hongmaju.github.io.git #这里的网址填你自己的
    branch: master
```
* 部署
```python
hexo d
```
这时再刷新 https://hongmaju.github.io 就可以看到你的博客了。