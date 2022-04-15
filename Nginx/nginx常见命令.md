

nginx常用命令：

```xml
nginx -v #显示版本信息并退出
nginx -V #显示版本和配置选项信息，然后退出
nginx -t #检测配置文件是否有语法错误，然后退出
nginx -T #检测配置文件是否有语法错误，转储并退出
nginx -s reopen #重启Nginx
nginx -s reload #重新加载Nginx配置文件，然后以优雅的方式重启Nginx
nginx -s stop #强制停止Nginx服务
nginx -s quit #优雅地停止Nginx服务，处理完所有请求后再停止服务
```

