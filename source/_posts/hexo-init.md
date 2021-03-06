---
title: Hexo Init
date: 2020-07-06 09:55:18
tags:
categories: 杂项
---
## 前提准备

 [Node.js](https://nodejs.org/en/)  
 [Git](https://git-scm.com/)

## Hexo 安装

```bash
npm install -g hexo
```

## Hexo 初始化

新建一个文件夹 (例如 Hexo-Blog) 进入并初始化

```bash
mkdir Hexo-Blog
cd Hexo-Blog
hexo init
```

这个文件夹必须为空 否则无法初始化成功。当要删除文件夹中的`node_modules`子文件夹时要安装 ``rimraf``

```bash
npm install rimraf -g
rimraf node_modules
```

## 修改主题并应用

在当前文件夹 下载新主题

```bash
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

修改 `Hexo-Blog\_config.yml`文件中`theme`：`landscape`改为`next`

修改 `themes\next\_config.yml` 中 `scheme`: `Muse` 改为 `Gemini`

在`themes\next\_config.yml` 中 `menu`取消标签和分类的注释

```json
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
```

### 更新Next主题

- 记得把`themes\next\_config.yml` 中:`mathjax`的`enbale`设置为`true`

```bash
hexo g #生成文件
hexo s #启动服务
```

`hexo s` 是开启本地预览服务，或者直接`hexo s -g`生成并预览，打开浏览器访问 [localhost:4000](http://localhost:4000) 即可看到内容

## 部署到github

复制 `user\.ssh\id_rsa.pub` 到github账户的 `settints` - `SSH keys`

```json
deploy:
  type: git ##注意 这里有空格
  repository: git@github.com:lclee0577/lclee.github.io.git
  branch: master
```

部署到github前要先安装hexo-deployer-git

```bash
npm install hexo-deployer-git  --save
```

```bash
hexo clean 
hexo d -g
```

## 异地管理

创建hexo分支 并设置为默认分支

```bash
cd Hexo-Blog 
git init
git branch hexo
git remote add origin https://github.com/lclee0577/lclee0577.github.io.git
```

在 github 网页端 `Settings` - `Branches` -`Default branch` 中将默认分支设置为`hexo` 并点击`update`确认。

异地管理只需克隆hexo分支，安装开发依赖，再编辑即可。

```bash
npm install --devDependencies
```

## 插入markdown目录

- 无需安装hexo-toc 使用next主题会根据不同级别的标题自动生成目录
- 在`next\_config.yml`中设置`toc` 的 `enable` 和 `number`即可

  ```yml
  toc:
    enable: true
    # Automatically add list number to toc.
    number: false
  ```

## 插入Latex

  <https://www.jianshu.com/p/7ab21c7f0674>
