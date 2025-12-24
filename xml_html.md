xml,html工具

# xmllint

```
curl 'https://ip.cn/ip/14.213.21.203.html'   --compressed   -H 'User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:144.0) Gecko/20100101 Firefox/144.0'   -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8'   -H 'Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2'   -H 'Accept-Encoding: gzip, deflate, br, zstd'   -H 'Referer: https://ip.cn/ip/14.213.21.203.html'   -H 'Connection: keep-alive'    -H 'Upgrade-Insecure-Requests: 1'   -H 'Sec-Fetch-Dest: document'   -H 'Sec-Fetch-Mode: navigate'   -H 'Sec-Fetch-Site: same-origin'   -H 'Sec-Fetch-User: ?1'   -H 'Priority: u=0, i'   -H 'Pragma: no-cache'   -H 'Cache-Control: no-cache' | xmllint --html --xpath '//span[@id="tab0_address"]/text()' -
```

最后的-不能删除，这个表示从stdin中读取内容。

https://devhints.io/xpath  xpath文档

# ipv6

```
curl 'https://ipconfig.com/api/ip-query'   --compressed   -X POST   -H 'User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:146.0) Gecko/20100101 Firefox/146.0'   -H 'Accept: */*'   -H 'Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2'   -H 'Accept-Encoding: gzip, deflate, br, zstd'   -H 'Referer: https://ipconfig.com/'   -H 'Content-Type: application/json'   -H 'Origin: https://ipconfig.com'   -H 'Alt-Used: ipconfig.com'   -H 'Connection: keep-alive'      -H 'Sec-Fetch-Dest: empty'   -H 'Sec-Fetch-Mode: cors'   -H 'Sec-Fetch-Site: same-origin'   -H 'Priority: u=0'   --data-raw '{"ip":"2408:8459:4ec0:3737:bcbb:5eff:fe27:9a24","lang":"en"}'
{"success":true,"data":{"ip":"2408:8459:4ec0:3737:bcbb:5eff:fe27:9a24","ip_start":"2408:8459:4ec0::","ip_end":"2408:8459:4fff:ffff:ffff:ffff:ffff:ffff","continent":"Asia","country":"China","province":"Guangdong","city":"Foshan","district":"Nanhai","isp":"China Unicom","longitude":"113.14299","latitude":"23.02877","area_code":"440605","city_code":"0757","zip_code":"528200","time_zone":"Asia/Shanghai","currency":"CNY","asn":"AS17816","elevation":"9","weather_station":"CHXX0028"},"meta":{"isLoggedIn":false,"isApiRequest":false,"remainingCount":8,"ipRemaining":8,"sessionRemaining":20,"limitedBy":"ip","dailyLimit":20,"sessionLimit":20}}
```

