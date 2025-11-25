xml,html工具

# xmllint

```
curl 'https://ip.cn/ip/14.213.21.203.html'   --compressed   -H 'User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:144.0) Gecko/20100101 Firefox/144.0'   -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8'   -H 'Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2'   -H 'Accept-Encoding: gzip, deflate, br, zstd'   -H 'Referer: https://ip.cn/ip/14.213.21.203.html'   -H 'Connection: keep-alive'    -H 'Upgrade-Insecure-Requests: 1'   -H 'Sec-Fetch-Dest: document'   -H 'Sec-Fetch-Mode: navigate'   -H 'Sec-Fetch-Site: same-origin'   -H 'Sec-Fetch-User: ?1'   -H 'Priority: u=0, i'   -H 'Pragma: no-cache'   -H 'Cache-Control: no-cache' | xmllint --html --xpath '//span[@id="tab0_address"]/text()' -
```

最后的-不能删除，这个表示从stdin中读取内容。

https://devhints.io/xpath  xpath文档