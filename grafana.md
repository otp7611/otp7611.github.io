grafana监控平台

# 整体结构

node_exporter -> prometheus -> grafana

# node_exporter

下载node_exporter-1.9.0.linux-amd64.tar.gz，运行./node_exporter

这个服务负责采集系统信息。

服务成功运行后会监听端口9100。

通过curl http://localhost:9100/metrics可能拿到采集的信息。

例如：

```
node_cpu_seconds_total{cpu="2",mode="idle"} 3.16436534e+06
```

node_cpu_seconds_total是metric,可以理解为事件名，测量项

cpu="2",mode="idle"是label, 可以理解为属性，

3.16436534e+06是事件值，测量值。

# prometheus

下载prometheus-2.53.3.linux-amd64.tar.gz, 运行./prometheus

这个服务负责收集和存储测量项。以时间序列的方法存储。这个工具本身是用来监控系统和对异常进行报警的。

服务成功运行后会监听端口9090。

如果想连接node_exporter，需要在配置文件prometheus.yml中增加：

```
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
  
  # connect to node_exporter
  - job_name: "node"
    static_configs:
      - targets: ['localhost:9100']

```

## 查看收集到的数据

访问http://localhost:9090/graph

如果想查每个cpu核的使用率，输入

```
 (rate (node_cpu_seconds_total{mode="idle"}[2m]))
```

# grafana

这个服务负责把测量项以图表的方式显示出来。

服务成功运行后会监听默认3000端口。

访问http://localhost:3000

## 连接prometheus

grafana支持多个数据库，如果想要连接prometheus，那就需要添加一个数据源prometheus。

http://localhost:3333/connections/add-new-connection

配置好Prometheus server URL为prometheus的服务地址：http://192.168.40.118:9090

不用配置Authentication，因为prometheus默认没有认证。

## 以容器的方式安装

### docker-compose.yaml

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
      - '/home/data/workspace/grafana/data:/data'
    environment:
      - TERM=screen-256color
volumes:
  grafana-storage: {}
```

docker的挂载方式bind mounts的权限与主机是一样的。如果主机的用户是userA那么在容器里面这个目录的所有者也是userA。而在容器里，默认权限是root。如果这个目录的权限other是没有权限的，那么容器的root也是不能使用这个目录的。

### 以root身份进入容器

```shell
sudo docker exec --user root  -it grafana /bin/bash
```

## 对数据作图

首先选择一个dashboard, 没有则创建。http://localhost:3333/dashboards

在dashboard里，进入edit模式，然后add->Visualization,就是增加一个可视图化。

然后就是进行配置。

### 选择数据源prometheus

### 配置查询语句

配置有两种方式builder和code, 选code模式这样就是直接输入表达式了。

比如查看cpu的使用率

```
 100*(1-(rate (node_cpu_seconds_total{mode="idle"}[2m])))
```





