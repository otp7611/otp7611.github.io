android

# adb无线连接

## 下载最新版本的adb

https://developer.android.google.cn/tools/releases/platform-tools?hl=zh-cn

## 与手机配对

```
adb pair <ip>:<port>
```

ip,port是配对ip和配对port

会提示输入配对码.输入后就连接了.

```
Enter pairing code:
```

## 处理error: unknown host service问题

可能是不同版本的adb占用了5037端口.找出关闭它就可以了.

 ```
 netstat -atpnl | grep 5037
 ```

