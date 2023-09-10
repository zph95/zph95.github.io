---
title:  "markdown添加动图和音频以及链接视频方式"
date:   2023-09-10 11:12:00 +0800
tags: 
    - markdown
    - jekyll
    - gitub pages
    - bilibili
toc: true 
toc_label: "Contents Table" 
toc_icon: "cog"
---

## gif, mp3, bilibili

最近目标： 打打球，练练琴，找找工作


### gif 

感觉自己再不努力一点就没得救了。

![gif_test](../../assets/gif/没得救了.gif)

### mp3

音频文件可以用 HTML 的方式嵌入到 Markdown 中。

使用 ```HTML5 <audio>``` 元素
下面是一个将音频嵌入到 HTML 文档的例子。
```html
<audio src="/test/audio.ogg">
  你的浏览器不支持 audio 标签。
</audio>
src 属性可以设置为一个音频文件的 URL 或者本地文件的路径。

<audio src="audio.mp3" preload="none" controls loop>
  你的浏览器不支持 audio 标签。
</audio>
```
这个例子的代码中使用了 HTML 的“audio”元素的一些属性：

- ‘‘controls’’ : 为网页中的音频显示标准的 HTML5 控制器。
- ‘‘loop’’ : 使音频自动重复播放。
- ‘‘preload’’ : 属性用来缓冲 audio 元素的大文件，有三个属性值可供设置：
- “none” 不缓冲文件
- “auto” 缓冲音频文件
- “metadata” 仅仅缓冲文件的元数据

<audio controls>
  <source src="../../assets/audio/遇见.mp3" type="audio/mp3">
</audio>

### mp4
原生仅支持播放 ogg(ogv/ogm)/mp4/webm 格式。不支持播放 FLV 格式视频。

在 HTML 中嵌入视频：
```html
<video width="320" controls loop>
  <source src="myVideo.mp4" type="video/mp4">
  <source src="myVideo.webm" type="video/webm">
  <source src="myVideo.ogv" type="video/ogg" />
  <p>Your browser doesn't support HTML5 video. Here is
     a <a href="myVideo.mp4">link to the video</a> instead.</p>
</video>
```
属性：

- ‘‘controls’’ : 允许用户控制视频的播放，包括音量，跨帧，暂停/恢复播放。
- ‘‘width’’ : 视频显示区域的宽度，单位是CSS像素。
- ‘‘height’’ : 展示区域的高度，单位是CSS像素。
- ‘‘loop’’ : 布尔属性；指定后，会在视频结尾的地方，自动返回视频开始的地方。
- ‘‘src’’ : 要嵌到页面的视频的URL。可选；你也可以使用video块内的 <source> 元素来指定需要嵌到页面的视频。


### 嵌入网站
```<iframe>``` 标签能够将另一个 HTML 页面嵌入到当前页面中。

嵌入网络视频：可以通过B站网页版点分享获取嵌入代码
```html
<iframe src="//player.bilibili.com/player.html?aid=830776358&bvid=BV1934y1N7ve&cid=1263064622&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
```

略加调整自适应窗口，得到如下。

```html
<div style="position: relative; width: 100%; height: 0; padding-bottom: 75%;">
    <iframe src="//player.bilibili.com/player.html?aid=830776358&bvid=BV1934y1N7ve&cid=1263064622&p=1"  scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="position: absolute; width: 100%; height: 100%; left: 0; top: 0;"></iframe>
</div>
```

<div style="position: relative; width: 100%; height: 0; padding-bottom: 75%;">
    <iframe src="//player.bilibili.com/player.html?aid=830776358&bvid=BV1934y1N7ve&cid=1263064622&p=1"  scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="position: absolute; width: 100%; height: 100%; left: 0; top: 0;"></iframe>
</div>



