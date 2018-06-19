---
title: HEXO 常用指令
date: 2018-05-22 19:52:51
toc: true
categories: 工具
tags: [Hexo,效率工具]
---
HEXO 常用指令 & 简单创建
<!-- more -->
### 官网
- 地址  

```
https://hexo.io/zh-cn/ 
```

### 初始化工程
- 初始化

```
hexo init MyBlog
```

### 本地预览
- 安装hexo server：

```
sudo npm install hexo-server
```
- 生成静态页面：

```
hexo g
```
- 打开hexo本地服务(指定端口5000，默认4000)：

```
hexo server -p 5000
```

### git部署
- 安装hexo的git部署：

```
npm install hexo-deployer-git --save
```
- 生成静态页面并部署到github：

```
hexo d -g（或hexo g + hexo deploy）
```

### 写新文章
- 新文章：

```
hexo new [layout] <title>
```
### 引用本地资源
- 安装插件：

    ```
    npm install hexo-asset-image --save
    ```
- 在_config.yml中修改设置：


    ```
    post_asset_folder: true
    ```
- 在abcd.md文章中引用：


    ```
    ![你想输入的替代文字](abcd/图片名.jpg)
    ```

### 添加RSS
- 安装插件：

```
npm install hexo-generator-feed --save
```
- 修改_config.yml：

```
feed:
    type: atom
    path: atom.xml
    limit: 20
```

### 搜索引擎收录
- 安装插件：

```
npm install hexo-generator-sitemap --save
```
- 修改_config.yml：

```
sitemap:
    path: sitemap.xml
```

### 百度搜索引擎收录
- 安装插件：

```
npm install hexo-generator-baidu-sitemap --save
```
- 修改_config.yml：

```
baidusitemap:
    path: baidusitemap.xml
```