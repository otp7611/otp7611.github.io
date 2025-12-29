tinyproxy

# 正向代理

```
upstream http 192.168.40.96:6666
upstream none "www.gosct.cn:3443"
```

如果你配置了upstream那么就一定要配置upstream none才能正常使用正向代理模式，否则它只会把请求发送到另一个代理服务器。

```
ConnectPort 443
ConnectPort 563
ConnectPort 3443
```

如果要正代理https就一定要配置ConnectPort，因为https会使用CONNECT方法。只有配置了ConnectPort才能正常处理CONNECT请求。

```
Allow 127.0.0.1
Allow ::1
Allow 192.168.0.0/16
```

这里需要把的允许的网段配置上。