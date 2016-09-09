---
layout:     post
title:      "在win10 环境搭建jekyll环境"
subtitle:   "jekyll 构建笔记"
date:       2016-09-08
author:     "vcoolwind"
header-img: "img/post-bg-js-module.jpg"
tags:
    - 环境构建
    - windows
---

# 在windows10 环境搭建jekyll环境
    话不多说，直接开整。

- 下载ruby，由于是windows环境，直接rubyinstaller进行安装。点[这里下载](http://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-2.3.1-x64.exe)现在最新的版本（rubyinstaller-2.3.1-x64.exe）。当然，如果你是32位机器，请下载对应的32位版本，[这里](http://rubyinstaller.org/downloads/)可以查看所有的版本。
- **安装ruby**。详细过程不再赘述，推荐选中`Add Ruby executables to your PATH` ,便于后续使用。
- 下载rubygems，最新的版本是2.6.6，地址在[这里](https://rubygems.global.ssl.fastly.net/rubygems/rubygems-2.6.6.zip)。 
- **安装rubygems**。解压rubygems的zip文件，进入对应的目录，执行`ruby setup.rb`。
- **安装jekyll**，执行`gem install jekyll -v 3.1.6`。泡杯茶，耐心等待一会。
- **测试验证**。

    ```
    1. jekyll new test
    2. cd test
    3. jekyll serve  #启动服务
    4. 嗯，服务如期的启动起来了。 可以看到对应的效果 -- http://127.0.0.1:4000/ 
    ```
- 搞一个模板来玩玩。看到有几个同学推荐[黄玄](https://github.com/Huxpro/huxpro.github.io)同学的模板，下载体验下。

    ```
    1. git clone https://github.com/Huxpro/huxpro.github.io.git
    2. gem install jekyll-paginate  #安装相应的依赖，根据个人需要，可以安装其他模板。 jemoji jekyll-feed jekyll-sitemap
    3. jekyll serve #跑起来
    4. 嗯，效果还不错，这就考虑先用这个模板喽。
    
    ```



### 过程中发现的问题及解决
- 使用`jekyll new test`建立的测试工程，无法通过`jekyll serve`启动。

    ``` 
    #jekyll -v
    jekyll 3.2.1
    #jekyll new test
    New jekyll site installed in C:/tmp/test.
    #jekyll serve
    C:/Ruby23-x64/lib/ruby/site_ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require': cannot load such file -- bundler (LoadError)
            from C:/Ruby23-x64/lib/ruby/site_ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
            from C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/jekyll-3.2.1/lib/jekyll/plugin_manager.rb:34:in `require_from_bundler'
            from C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/jekyll-3.2.1/exe/jekyll:9:in `<top (required)>'
            from C:/Ruby23-x64/bin/jekyll:22:in `load'
            from C:/Ruby23-x64/bin/jekyll:22:in `<main>'
    ```
- [ ] 解决方法：[这里](https://teamtreehouse.com/community/im-trying-to-get-jekyll-working-on-windows-10-help)发现有同样的人提问，采用类似的办法，版本回退到3.1.6解决。jellky也说了没有官方的支持，果不欺我。