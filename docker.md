docker

# 启动已经退出的容器

```
docker start -i <containerID>
docker start <containerID>
```

docker start 带参数-i表示交互模式，如果docker start停止，则容器停止。如果不带，则会在后台启动这个容器。

# 复制已经退出容器里的文件

```
docker cp -a <containerID>:/usr/share/elasticsearch/config/jvm.options   ./
docker cp jvm.options <containerID>:/usr/share/elasticsearch/config/jvm.options
```

# docker在主机上与容器相关的配置，日志，状态文件

```
/var/lib/docker/containers/<containerID>/
```

# 更新容器内存限制

```
sudo docker update --memory "5g" --memory-swap "-1" <containerID>
```

