---
title: 将本地Hexo站点备份到Github上
date: 2021-02-24 16:58:07
tags:
    - 教程
---

## 一、本地配置
由于是在个人电脑上配置的hexo，部署也不方便，如果在另外一台机子上写博客，没有hexo的配置也不行，另一方面出于备份项目的目的，最好的办法是将本地hexo站点备份到Github上。
1.新建`hexo`文件夹存放分支工作目录。
``` bash
    mkdir hexo
```
2.把你的GitHub的远程仓库克隆到`hexo`文件夹
``` bash
	git clone https://github.com/HK560/hk560.github.io hexo
```
3.删除除了版本管理的`.git`之外的所有文件和文件夹
``` bash
    cd hexo
    rm -r *
```
4.把要备份的文件复制到`hexo`目录
> scaffolds/
source/
themes/
.gitignore
_config.yml
package.json

5.如果使用的主题是从Github克隆的，那么使用命令删除它的Git文件（以next主题为例）,否则无法将主题文件`push`
``` bash
rm -R themes/next/.git*
```
## 二、github配置
1.github创建一个`hexo`分支
``` bash
git checkout -b hexo
```
2.保存所有文件到暂存区
``` bash
git add --all
```
3.提交变更
``` bash
git commit -m "创建hexo分支"
```
4.推送到github，并用`--set-upstream`与`origin`创建关联，将`hexo`设置为默认分支
``` bash
git push --set-upstream origin hexo
```
## 三、合并管理
现在有两个需要管理的文件夹，一个是`blog`，一个是`hexo`，但是每次发布文章的时候如果要提交两次就很失效率，
1.将本地`hexo`分支中的`.git`文件夹复制到`blog`根目录中
``` bash
cp -a .git ../blog
```
2.`master`分支的文件则由`hexo`管理，编辑`hexo`配置文件`_config.yml`
```
deploy:
   type: git
   repo: https://github.com/HK560/hk560.github.io
   branch: master
```

## 四、发布文章
1.新建文章
``` bash
hexo new test
```
2.将相关更改推送到hexo分支
```bash
git add .
git commit -m "发表文章test"
git push origin hexo
```
3.将静态文件推送到master分支
```bash
hexo clean 
hexo d -g
```

## 五、使用的时候如何迁移
1.将hexo分支克隆下来
``` bash
git clone -b hexo https://github.com/HK560/hk560.github.io
```
2.安装nodejs，设置好镜像站

3.安装hexo
``` bash
npm install -g hexo-cli
```

4.安装hexo依赖
cd到仓库根目录
``` bash
npm install
```
只不过每次发文章的时候，要输入用户名和密码。


> 参考自：[将本地hexo站点备份到github上](https://blog.csdn.net/weixin_44861399/article/details/104936907 "将本地hexo站点备份到github上")