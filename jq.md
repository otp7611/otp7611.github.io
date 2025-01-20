json串解析工具jq

如果是json格式的文件，后缀用 .json可以在命令行中自动补齐。

# 计算数组长度

```
jq length a.json
```

# 其它

```
echo '{"conf":"{\"a\":1}","result":0}' | jq '.conf | fromjson'
jq '.streams[] | select(.sid | contains("field")) | .' a.info | jq '[inputs]'
curl 'http://127.0.0.1/some' | jq '.servers[] | select(.type=="thisistype")'
jq . <<<$(cat conf/mediamux.conf) > conf/mediamux.conf

```

