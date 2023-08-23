---
title:  "dubbo-admin的使用记录和改造内容"
date:   2023-07-19 23:19:00 +0800
tags: 
    - dubbo-admin
    - nacos
toc: true 
toc_label: "Contents Table" 
toc_icon: "cog"
---

## dubbo admin 
dubbo-admin的功能挺多，就是服务关系显示的不好看。将力型图改成了环状图, 可高亮和缩放，显示了使用的dubbo版本。fork的分支地址：
[dubbo-admin_0.5.0](https://github.com/zph95/dubbo-admin)
![relation](relationV2.png)
## dubbo3.0 注册中心找不到消费者地址原因

### 注册接口级消费者 
Dubbo3.0.0版本以后，增加了是否注册消费者的参数，如果需要将消费者注册到nacos注册中心上，需要将参数(register-consumer-url)设置为true，默认是false。

#### application.yml
```yml
dubbo:
  registry:
    address: nacos://localhost:8848?register-consumer-url=true
```
或者
### application.yml
```yml
dubbo:
  registry:
    address: nacos://localhost:8848
    parameters.register-consumer-url: true
```

