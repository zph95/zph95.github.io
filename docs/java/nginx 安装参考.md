
# 如何使用nginx作为文件服务器 

Nginx("engine x")是一款是高性能的 Web和 反向代理 服务器，也是一个 IMAP/POP3/SMTP 代理服务器。在高连接并发的情况下，Nginx是Apache服务器不错的替代品。

## 安装方式

yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel

- rz上传文件安装包,解压，nginx-upload-module模块,nginx-upload-progress-module（上传进度模块，可不安装）

```
cd nginx-1.18.0
 ./configure --add-module=../nginx-upload-module-2.3.0 --add-module=../nginx-upload-progress-module-0.9.2
make
make install
```

- nginx.conf文件添加配置

```nginx
location /download {
            alias /tmp/upload_file/;
            autoindex on;
            autoindex_exact_size off;
            autoindex_localtime on;
        }

location /upload {  
    upload_pass     /;  
    # upload_cleanup 400 404 499 500-505;  
    upload_store    /tmp/upload_file/;  
    upload_store_access user:rw;  
    # upload_limit_rate 128k;  
    upload_set_form_field "${upload_field_name}_name" $upload_file_name;  
    upload_set_form_field "${upload_field_name}_content_type" $upload_content_type;  
    upload_set_form_field "${upload_field_name}_path" $upload_tmp_path;  
    upload_aggregate_form_field "${upload_field_name}_md5" $upload_file_md5;  
    upload_aggregate_form_field "${upload_field_name}_size" $upload_file_size;  
    upload_pass_form_field "^.*$";  
}
```

## 一些命令

关闭nginx进程
nginx -s stop
启动nginx进程
/usr/sbin/nginx          yum安装的nginx也可以使用         servic nginx start
检查配置文件是否有误
nginx –t
重新加载配置文件
nginx –s reload




# 问题：在上传较大的pin包文件时会失败，业务服务器接收不到上传的数据。

是由于nginx对上传文件的大小有限制，默认是1M，另外如果文件过大导致后端处理时间过长，nginx会等待超时中断请求，所以要将超时时间配置大一些，以便于后端能将文件处理完毕。

解决：修改nginx配置文件，client_max_body_size这个参数的默认值为1M，我将其修改为50M，可以上传1000w条pin。proxy_read_timeout这个参数默认值是60s，将其修改为10分钟。
http{} 中控制着所有nginx收到的请求。而报文大小限制设置在server｛｝中，则控制该server收到的请求报文大小，同理，如果配置在location中，则报文大小限制，只对匹配了location 路由规则的请求生效。

# 其它
nginx_upload_module模块用于接受并处理用户上传的文件，网上有很多该模块的配置方式。文件上传功能涉及webshell安全问题，其实一般不建议直接使用nginx来上传文件，而是通过后端服务来处理文件上传，nginx只为后端服务做一个简单的代理，后端服务先对访问用户进行鉴权，再将上传文件存放在非web目录。nginx upload上传的文件会以临时文件的方式存在路径（/tmp/upload_file/）中，文件名是文件上传的顺序编号，不带文件后缀。其中关于文件的信息会作为参数传到后续处理程序中，后续程序可以使用多种方式实现。由此看来，nginx_upload上传只是做了一层缓存文件转发的操作，最终还是要专门的程序处理文件操作。所以，nginx配置下载服务可以作为简单文件下载服务器，上传文件还是需要使用专门的文件对象存储服务。