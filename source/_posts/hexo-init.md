---
title: Hexo Init
date: 2020-07-06 09:55:18
tags:
---
## 前提准备
 [Node.js](https://nodejs.org/en/)  
 [Git](https://git-scm.com/)

## Hexo 安装
```bash
$ npm install -g hexo
```
## Hexo 初始化
新建一个文件夹 (例如 Hexo-Blog) 进入并初始化

```bash
$ mkdir Hexo-Blog
$ cd Hexo-Blog
$ hexo init
```
这个文件夹必须为空 否则无法初始化成功。当要删除文件夹中的`node_modules`子文件夹时要安装 ``rimraf``

```bash
$ npm install rimraf -g
$ rimraf node_modules
```

## 修改主题并应用

在当前文件夹 下载新主题
```bash
$ git clone https://github.com/theme-next/hexo-theme-next themes/next
```
修改 `Hexo-Blog\_config.yml`文件中`theme`：`landscape`改为`next` 

修改 `themes\next\_config.yml` 中 `scheme`: `Muse` 改为 `Gemini`
```bash
$ hexo g #生成文件
$ hexo s #启动服务
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
$ npm install hexo-deployer-git  --save
```
```bash
$ hexo clean 
$ hexo d -g
```

## 异地管理
创建hexo分支 并设置为默认分支
```bash
$ cd Hexo-Blog 
$ git init
$ git branch hexo
$ git remote add origin https://github.com/lclee0577/lclee0577.github.io.git

```
在 github 网页端 `Settings` - `Branches` -`Default branch` 中将默认分支设置为`hexo` 并点击`update`确认。

异地管理只需克隆hexo分支再编辑即可。

## 插入markdown目录
 - npm install hexo-toc --save
 - 配置文件_config.yml:
 - ```json
      toc:
        maxdepth: 3
        class: toc
        slugify: transliteration
        decodeEntities: false
        anchor:
          position: after
          symbol: '#'
          style: header-anchor
      ```

- 使用`<!-- toc -->` 将自动生成目录
- 文章最上面与标题一起处添加 `toc: true `

## 插入Latex

  https://www.jianshu.com/p/7ab21c7f0674