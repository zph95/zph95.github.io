---
layout: post
title:  "使用 Githup pages 搭一个博客网站"
date:   2021-04-01 22:29:36 +0800
categories: jekyll
typora-root-url: ..
---

# 如何搭一个博客网站?

如果只想简单的写写文章，可以选择微信公众号，简书，知乎专栏，博客园，CSDN等网站提供的服务。	

当如果想从零开始搭一个网站，可要做的事情就比较多了。首先是买服务器，还要买域名，还要自己学WordPress等工具。

折中一下，使用Github Pages搭一个静态网站，就简单多了。

### 创建一个Github Pages repository

GitHub Pages repository跟普通的repository是一样的，唯一的区别就是它的名字必须叫做username.gihub.io。这个官方教程 [GitHub Pages](https://pages.github.com) 写的十分好懂，按这个做完之后你就有了一个你的网址 username.github.io，里面会有一个基础的readme.md文件和_config.yaml文件。

## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/zph-programmer/zph-programer.github.io/edit/main/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/zph-programmer/zph-programer.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.



## Github Pages 的插件

github pages 使用默认启用了以下插件，并且无法从页面上禁用。你可以在*_config.yml*文件中启用额外的插件。 

- [`jekyll-coffeescript`](https://github.com/jekyll/jekyll-coffeescript)
- [`jekyll-default-layout`](https://github.com/benbalter/jekyll-default-layout)
- [`jekyll-gist`](https://github.com/jekyll/jekyll-gist)
- [`jekyll-github-metadata`](https://github.com/jekyll/github-metadata)
- [`jekyll-optional-front-matter`](https://github.com/benbalter/jekyll-optional-front-matter)
- [`jekyll-paginate`](https://github.com/jekyll/jekyll-paginate)
- [`jekyll-readme-index`](https://github.com/benbalter/jekyll-readme-index)
- [`jekyll-titles-from-headings`](https://github.com/benbalter/jekyll-titles-from-headings)
- [`jekyll-relative-links`](https://github.com/benbalter/jekyll-relative-links)

## 简单发布一篇新的文章

jekyll-readme-index会将readme.md文件作为初始的index文件，所以当你从http://username.github.io进入你的个人网站时，首先看到的会是jekyll将readme.md生成的html页面(配上你选择的主题样式)。这个页面的内容可以清掉，作为博客的目录。如果你还不熟悉markdown语法，可以将原readme.md备份成readme_copy.md。

markdown基本语法可以参考：

[Markdown 语法说明(简体中文版) (appinn.com)](https://www.appinn.com/markdown/)

jekyll-relative-links可以实现生成网页之间通过相对路径跳转，你可以在readme.md文件中写下

```
[foo1](bar1.md)
[foo2](docs\bar2.md)
```

然后，就可以在从http://username.github.io页面，点击foo1跳转到http://username.github.io/bar1.html了，点击foo2跳转http://username.github.io/docs/bar2.html。



## Github Pages的使用限制

Github Page对jekyll的支持是很到位的, 唯一的不足可能也是其本身基于安全考虑而使得jekyll始终都是运行在safe模式, 目前放开的插件列表非常有限, 所以很多jekyll的插件都无法使用. 

GitHub Pages sites are subject to the following usage limits:

- GitHub Pages source repositories have a recommended limit of 1GB. For more information, see "[What is my disk quota?"](https://docs.github.com/en/articles/what-is-my-disk-quota/#file-and-repository-size-limitations)
- Published GitHub Pages sites may be no larger than 1 GB.
- GitHub Pages sites have a *soft* bandwidth limit of 100GB per month.
- GitHub Pages sites have a *soft* limit of 10 builds per hour.

 
### jekyll plugin
 作为一个流行的静态blog, jekyll的社区和支持者也是非常众多的, 大家可以在github上搜索jekyll就能找到很多jekyll的插件了. 

### 集成plugin的jekyll与github page
  在github page不支持插件的情况下, 我们该如何选择呢?
  思路参考：https://taoalpha.github.io/blog/2015/05/29/tech-use-jekyll-plugin-with-github-page/

换: github page主要是因为安全因素而强迫jekyll服务必须在safe下运行, 那么我们换一个服务器的话自然就完全由我们自己可控了, 或者换一个支持jekyll的公共服务即可;

推: github page对jekyll的支持, 本质上还是对静态网页的支持, 所以如果我们在本地编译好jekyll然后把build后的_site文件夹推送到github page上也是肯定可以的;

绕: 如果你觉得每次这么推送比较痛苦, 而且还是想要把jekyll部分的代码也放在github上的话, 那么可以考虑用一个绕一些的办法, 通过github本身支持project page, 结合推的办法, 我们就可以新建一个repo, 然后在master分支管理原始代码, 在gh-pages分支存放生成的site代码. 然后通过xxx.github.io/repo-name来访问了.


