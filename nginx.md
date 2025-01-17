使用nginx代理服务器

官方文档https://nginx.org/en/docs/

# nginx预定义的变量

https://nginx.org/en/docs/http/ngx_http_core_module.html#variables

$uri current URI in request, normalized

http请求可以表示为：

```nginx
$scheme://$host$uri$is_args$args
$scheme://$host$request_uri
```

# 路径匹配location

参考文档：https://nginx.org/en/docs/http/ngx_http_core_module.html#location

nginx匹配都是部分匹配。location前缀匹配和正则匹配。前缀匹配是从开始匹配部分，正则配置则是搜索匹配。

## 多个匹配时，location的选择

1. "="修饰符，完整匹配
2. "^~"选择最长的前缀匹配，不再进行正则匹配。
3. "~","~*"第一个正则匹配，注意选则的不是最长匹配，是第一个正则匹配的location.
4. 无任何修饰符，选择最长的前缀匹配。

## 服务静态文件

```nginx
location /testm3u8 {
        root /data;
        index index.html;
        try_files $uri $uri/ =404;
        add_header Access-Control-Allow-Origin *;
    }
```

不要用正则表达式。

如果http请求是http://127.0.0.1/testm3u8/a.m3u8会响应/data/testm3u8/a.m3u8文件。即是root路径加上http的路径值。

## 自动截断path并把请求进行转发

```
location /127.0.0.1/ {
        proxy_pass http://127.0.0.1:8000/;
        proxy_http_version 1.1;
    }

```

如果请求为http://127.0.0.1/other.host.name/path?a=b那么会转发为http://other.host.name/path?a=b,这个请求会发到127.0.0.1:8000端口上。

注意，如果location没有以/结尾会出现301重定向。

注意，如果proxy_pass没有带uri（这里是/）,那么会把请求的完整uri传过去，否则location中的匹配部分（这里是/127.0.0.1/）会被替换成proxy_pass中的uri。就是http请求中的/127.0.0.1/替换成了/.

https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass

## 正则匹配路径

```
location ~ live.m3u8$ {
        root /data/stream_m3u8/https;
        add_header Access-Control-Allow-Origin *;
    }

```

开始正则匹配用~.这里响应的文件也是root路径加上http的路径值。

# 依据查询参数跳转

```nginx
server {
    listen 8080;
    # host
    server_name other.host.name;
    location /path {
        set $target_site http://127.0.0.1:8081;
        if ($arg_queryname ~* abcd.*) {
            set $target_site http://127.0.0.1:8082;
        }
        proxy_pass $target_site;
        proxy_http_version 1.1;
    }
}
```

这个配置意思是：

处理端口8080上收到域名为other.host.name的http请求,首部中Host字段是other.host.name:8080

如果请求是http://other.host.name:8080/path且查询参数queryname以abcd开头，则把http请求转发到http://127.0.0.1:8082，路径，查询参数等都会转发过去。否则转发到http://127.0.0.1:8081。

# 处理nginx无法响应本地文件的问题

如果文件确实存在，但是响应却异常。这一样是这个文件的权限有问题。这个权限包括目录的可访问权限。这个可以通过namei这个命令来解决。

```shell
namei -l /data/testm3u8/my.m3u8 
f: /data/testm3u8/my.m3u8
drwxr-xr-x root  root  /
lrwxrwxrwx root  root  data -> /media/workspace/data2
drwxr-xr-x root  root    /
drwxr-xr-x root  root    media
drwx---rwx chenc chenc   workspace
drwxrwxrwx chenc chenc   data2
drwxrwxrwx chenc chenc testm3u8
-rwxrwxrwx chenc chenc my.m3u8
```

从中可以看到，/data是一个链接。



