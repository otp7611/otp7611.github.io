docker

# 要使用docker image load引入压缩包

docker image load会引用启动命令，docker image import不会。

docker image load和docker image save对象都是压缩包，而docker image import和docker image export对象是容器。

```
docker image import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
docker tag f40343512527 grafana/grafana-enterprise:11.5.2-ubuntu
sudo docker image load --input  grafana-enterprise_11.5.2-ubuntu.tar
```

# 创建网络

同一网络下的容器能互连

```
docker network create elastic
```

# 创建磁盘卷

```
sudo docker volume create grafana-storage
```



# 使用制品启动容器

```
sudo docker run -dit --name grafana -e TERM=screen-256color -p 6800:3000 -v /data/video/grafana/sharedata:/data -v grafana-storage:/var/lib/grafana  grafana/grafana-enterprise:11.5.2-ubuntu
```

## 使用docker-compose.yaml

docker-compose.yaml

```
services:
  grafana:
    image: grafana/grafana-enterprise:11.5.2-ubuntu
    container_name: grafana
    restart: no
    ports:
      - '3333:3000'
    volumes:
      - 'grafana-storage:/var/lib/grafana'
      - '/home/data/grafana/data:/data'
    environment:
      - HTTPS_PROXY=
      - https_proxy=
      - http_proxy=
      - HTTP_PROXY=
      - ALL_PROXY=
      - all_proxy=
volumes:
  grafana-storage: {}
```

```
services:
  grafana:
    image: grafana/grafana-enterprise:11.5.2-ubuntu
    container_name: grafana
    entrypoint: /bin/bash
    command: []
    stdin_open: true
    tty: true
    restart: no
    # pid: host
    privileged: true
    cgroup: host
    deploy:
        resources:
            limits:
                memory: 5g
    volumes:
      - '/media/workspace/builder_data:/data'
    environment:
      - HTTPS_PROXY=
      - https_proxy=
      - http_proxy=
      - HTTP_PROXY=
      - ALL_PROXY=
      - all_proxy=
volumes:
  imaged-storage: {}
```

entrypoint command，command是命令的参数。

stdin_open tty使用伪终端。如果不使用这两个参数，bash启动后就退出了。

pid参数控制容器内的pid空间是否与主机空间一致。如果一致，如容器停止后会容器内的程序不会停止，他会被主机的1号进程接管。

privileged表示容器内用户权限，如果为true就不用有docker容器特定的权限限制。

cgroup表示容器内与主机一致，如果没有则容器内不能创建cgroup.

deploy控制部署上的约束。

## 常用命令

```
docker compose up -d
docker exec -it dockername /bin/bash
docker compose stop
docker compose rm
```

docker compose操作的是docker-compose.yaml里描述的所有服务。

-d表示在后台跑，否则为前台。如果ctrl-c则所有容器服务停止。

# 启动已经退出的容器

```
docker start -i <containerID>
docker start <containerID>
```

docker start 带参数-i表示交互模式，如果docker start停止，则容器停止。如果不带，则会在后台启动这个容器。

# 停止正在运行的容器

```
sudo docker stop es01
```

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

# 以root身份进入容器

```shell
sudo docker exec --user root  -it grafana /bin/bash
```

# 获取容器的信息

```shell
sudo docker inspect es01
```

# 查看容器的输出日志

```shell
sudo docker logs -t grafana
```

