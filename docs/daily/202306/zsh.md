# zsh

3.zsh——终端美化
这里使用的是Gitee上的一个开源zsh管理插件​ ​Tmoe-zsh​​​ ，基于​​ZINIT​​​工具开发，个人感觉启动速度和视觉延迟是要优于单纯的​​ohmyzsh​​​的，​​Tmoe-zsh​​​集成了很多功能，​​zsh​​​的安装卸载，备份还原比较方便。
安装命令：

```bash
pkg install curl && . <(curl -L l.tmoe.me/ee/zsh) -y
```

个人配置
|配置选项|zshfont|zshcolor|zshtheme|
|选择|Go Mono|Neon|powerlevel10k|
|选项数字|14|85|171|

安装结束输入：​​zsh​​ 使配置生效，效果：
如果对第一次配置结果不满意，使用以下命令可再次配置：

```bash
zshtheme    #修改zsh主题
zshcolor    #修改终端配色  
zshfont      #修改终端字体  
zsh-i      #打开Tmoe-zsh管理器
```

​​Tmoe​​​除了配置zsh字体主题外，还自动安装了一些高效插件：​ ​PLUGINS 预装插件说明​
