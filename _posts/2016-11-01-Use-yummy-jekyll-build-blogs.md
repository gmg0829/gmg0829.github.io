---
layout: post
title: 使用Yummy-Jekyll构建自己的博客
category: jekyll
tags: [jekyll, blog]
---

我的个人博客：<https://freemantc9527.github.io/>，欢迎围观。

## win7本地运行环境

1. 建立 Ruby 环境

   下载 `Ruby` 的Windows安装包，注意32位还是64位。安装时记得 `Add Ruby executables to your PATH`！安装完成后打开命令行，执行

   ```command-line
   ruby -v
   ```
   
   下载地址：<http://rubyinstaller.org/downloads/>

2. 安装 Ruby DevKit

   还是在刚才的下载页面，下载 `DEVELOPMENT KIT` 后，解压缩到任意目录，注意对应32位还是64位。打开命令行，转到DevKit解压后的目录，执行

   ```command-line
   ruby dk.rb init
   ```

   系统会在对应目录中生成 `config.yml` 文件，用文本编辑器打开，修改自己的ruby安装路径，如下：

   ```yml
    #
    # Example:
    #
    # ---
    # - C:/ruby19trunk
    # - C:/ruby192dev
    #
    ---
    - C:/Ruby23-x64
   ```

   修改完毕后，继续执行

   ```command-line
   ruby dk.rb install
   ```

3. 配置 RubyGems 的镜像 [gems.ruby-china.org](https://gems.ruby-china.org/)
   
   > 为什么有这个？  
   > RubyGems 一直以来在国内都非常难访问到，在本地你或许可以翻墙，当你要发布上线的时候，你就很难搞了！  
   > 这是一个完整 RubyGems 镜像，你可以用此代替官方版本，我们是基于国内 CDN + 国外服务器的方式，能确保几乎无延迟的同步。

   如果遇到 Windows 下证书无法验证问题 (certificate verify failed)，首先下载证书 <http://curl.haxx.se/ca/cacert.pem>，
   然后再环境变量里设置 `SSL_CERT_FILE` 这个环境变量，并指向 cacert.pem 文件。

   请参阅文档：<https://github.com/ruby-china/rubygems-mirror/wiki>

   执行

   ```command-line
   gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
   ```

   检查gem的源，确保只有 gems.ruby-china.org

   ```command-line
   gem sources -l
   ```

   更新完 `RubyGems` 之后，可以顺便做个升级 

   ```command-line
   gem update --system
   gem -v
   ```

4. 安装 Jekyll 

   ```command-line
   gem install jekyll
   ```

5. 安装 Bundler 

   ```command-line
   gem install bundler
   ```

## 项目修改

1. 下载 [Yummy-Jekyll](https://github.com/DONGChuan/Yummy-Jekyll) 
   
   可以用git clone的方式，也可以用fork的方式，当然更可以用下载zip的方式。
   
2. 修改配置 `_config.yml`

   文件里的名称和注释写的非常清楚了，按说明进行修改即可。Google Analytics 的部分可以全部注释掉（删除）。
   
   注意：`github_url` 和 `repository` 两个配置项是展示 github 上的项目用的，请填写真实地址，否则运行时会报异常。

3. 自定义修改

   根据自己的需要，修改相关对应文件，比如：about.md 

## 项目本地运行

1. 修改 Gemfile.lock 文件

   将其中 `nokogiri` 的版本号，由 1.6.7 修改为 1.6.8.1

   可以将 `remote` 项中的地址修改为：<https://gems.ruby-china.org/>

   Gemfile 文件中 `source` 项中的地址也可以修改为：<https://gems.ruby-china.org/>

2. 添加 bower_components 文件夹

   可以去 <https://github.com/DONGChuan/DONGChuan.github.io> 项目下载，或者使用 `bower` 来安装。

3. 运行

   打开命令行，进入项目对应目录，执行 

   ```command-line
   bundle install
   ```

   程序会根据Gemfile的配置，自动安装所缺失的依赖项。

   安装完依赖项之后，继续执行

   ```command-line
   bundle exec jekyll serve
   ```

   待命令提示行显示 `Server running... press ctrl-c to stop.` 的时候，可以打开浏览器访问了 <http://127.0.0.1:4000/>

## 致谢

感谢 [DONGChuan](http://dongchuan.github.io) 提供了这么好的模板 [Yummy-Jekyll](https://github.com/DONGChuan/Yummy-Jekyll) ！

感谢 [码志](http://mazhuang.org/) 提供了实践先例，让后来者走的更加轻松！

[1]: https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/   
[2]: https://github.com/ruby-china/rubygems-mirror/wiki   

