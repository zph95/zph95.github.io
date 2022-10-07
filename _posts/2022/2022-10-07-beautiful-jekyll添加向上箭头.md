---
layout: post
title:  "beautiful-jekyll添加向上箭头"
date:   2022-10-07 14:30:00 +0800
categories: js
typora-root-url: ..
---

## 参考
转载自：[Jekyll个人博客添加返回顶部按钮](https://zoharandroid.github.io/2019-08-04-Jekyll%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%B7%BB%E5%8A%A0%E8%BF%94%E5%9B%9E%E9%A1%B6%E9%83%A8%E6%8C%89%E9%92%AE/)

## 其它问题
beautiful-jekyll版本：5.0.0

1.没有main.css文件

 解决方案：
 新建一个/asset/css/main.css
 在base.html添加

```
common-css:
  - "/assets/css/main.css"
```

2.jQuery throw the error `fadeOut is not a function' 

    reason:

    This will happen if you're using the "slim" version of jQuery. Only the "full" version of jQuery includes animation effects.

    Try grabbing the "full" version of jQuery from the jQuery downloads page and including that in your page (or including a full version of jQuery from a CDN from your page).
    
解决方案：
将base.html中的jQuery url改成
```
common-ext-js:
  - href: "https://code.jquery.com/jquery-3.5.1.min.js"
    sri: "sha256-9/aliU8dGd2tb6OSsuzixeV4y/faTqgFtohetphbbj0="
```