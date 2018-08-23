---
layout: post
title: Markdown Reader 插件改造
category: chrome-extension
tags: [chrome, markdown]
---

Markdown Reader 是一款比较好用的浏览 markdown 文件的 chrome 插件。

安装地址：<https://chrome.google.com/webstore/detail/markdown-reader/gpoigdifkoadgajcincpilkjmejcaanc>

<link href="//cdn.bootcss.com/font-awesome/4.7.0/css/font-awesome.min.css" rel="stylesheet">

## 准备工作
1. 从应用商店安装扩展
1. 打开chrome插件管理（`chrome://extensions`）找到插件对应的ID
1. 从 `%USERPROFILE%\AppData\Local\Google\Chrome\User Data\Default\Extensions` 找到对应的目录
1. 将插件主体复制出来，删除其中的 `_metadata` 目录
1. 修改 `manifest.json` 文件，删除 `update_url` 项，修改 `web_accessible_resources` 项的内容为：` [ "*.*" ]`  
1. 选择chrome插件管理的 `开发者模式` ，并 `加载已解压的扩展程序...`
1. 勾选 `允许访问文件网址` 

## 改造一 <i class="fa fa-chrome"></i> ：链接新标签页（窗口）打开
修改 <i class="fa fa-file-code-o"></i> `markdownreader.js` 文件，在对应的样式加载代码后面，添加如下代码：

```javascript
var baseTarget = document.createElement('base');
baseTarget.target = '_blank';
document.head.appendChild(baseTarget);
```

## 改造二 <i class="fa fa-font-awesome"></i> ：添加 font awesome 图标支持
下载最新的 [<i class="fa fa-cloud-download"></i> Font Awesome 源码包](http://fontawesome.io/) ，解压缩后将文件放入工作目录。  
修改 <i class="fa fa-file-code-o"></i> `markdownreader.js` 文件，在对应的样式加载代码后面，添加如下代码：

```javascript
link = document.createElement('link');
link.rel = 'stylesheet';
link.href = chrome.extension.getURL('font-awesome-4.6.3/css/font-awesome.min.css');
document.head.appendChild(link);
```

## 改造三 <i class="fa fa-print"></i> ：修改打印样式 
修改 <i class="fa fa-file-code-o"></i> `markdownreader.css` 文件，在最后面，添加如下代码：

```css
@media print { 
	body{width: 21cm;margin:0;padding:0;}
	.content{
	  width: 88%;
	  background-color: #F8F8F8;
	  border:1px solid #ccc;
	  box-shadow:0 0 10px #999;
	  line-height:1.4em;
	  font-family: "PingFang SC", "Hiragino Sans GB", "Microsoft Yahei", helvetica, arial, freesans, clean, sans-serif;
      font-size:13.34px;
	  color:black;
	}
	#markdown-outline, #markdown-backTop, #markdown-outline ul, #markdown-outline ul:first-child, #markdown-outline li{
	  display: none; 
	  padding: 0;
	  margin: 0;
	  width:0;
	}
} 
```
