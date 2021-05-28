---
layout: post
title:  "how to install jekyll!"
date:   2021-05-27 14:36:36 +0800
categories: jekyll install
---

# Ubuntu下安装jekyll



## 准备工作

sudo apt install gcc g++ make

安装 Jekyll 相当简单，但是你得先做好一些准备工作。开始前你需要确保你在系统里已经有如下配置。

- [Ruby](http://www.ruby-lang.org/en/downloads/)（including development headers, Jekyll 2 需要 v1.9.3 及以上版本，Jekyll 3 需要 v2 及以上版本）

  - sudo apt install ruby bugy-dev

- [RubyGems](http://rubygems.org/pages/download)

  - sudo apt install rubygems

- Linux 

  - Ubuntu

- [NodeJS](http://nodejs.org/), 或其他 JavaScript 运行环境（Jekyll 2 或更早版本需要 CoffeeScript 支持）。

  sudo apt install node  npm

- [Python 2.7](https://www.python.org/downloads/)（Jekyll 2 或更早版本）

  sudo apt install python

## 用 RubyGems 安装 Jekyll[Permalink](http://jekyllcn.com/docs/installation/#用-rubygems-安装-jekyll)

安装 Jekyll 的最好方式就是使用 [RubyGems](http://rubygems.org/pages/download). 你只需要打开终端输入以下命令就可以安装了：

```
$ gem install jekyll
```

所有的 Jekyll 的 gem 依赖包都会被自动安装，所以你完全不用去担心

## 快速开始
  gem install jekyll bundler

  jekyll new my-awesome-site

  cd my-awesome-site

  bundle install

  bundle exec jekyll serve

# => 打开浏览器 http://localhost:4000