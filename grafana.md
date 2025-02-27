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

## 连接elasticsearch

在添加数据中选择elasticsearch。

connection url为：https://192.168.40.118:9200

Authentications与elasticsearch重置密码后参数一致。这个参数可以通过curl确定。

tls setting选择关闭证书验证Skip TLS certificate validation

Elasticsearch details参数中配置index name, pattern(设置为no pattern), Time field name(设置为@timestamp)

确认是否正常的命令：

```shell
curl -k -u elastic:$ELASTIC_PASSWORD -X GET "https://localhost:9200/testindex/_mapping?pretty"
```

应该会输出@timestamp，如果有@timestamp但是还是失败，请在kibana中discover一下数据内部，确认@timestamp是最近当前时间的毫秒整数。

### 作图

在添加监控图panel时，在raw视图能查到数据，但在metric视图不能作图。

- 要确认数据项是数字，不能是字符串之类，如果不能只能用聚合函数比如counter, unique_counter来进行数字化。
- 要确认数据源是连续的采样值，在一个时间间隔，如果只有一个时刻点的数据，是不能作图的。

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

### 查看grafana的输出日志

```
sudo docker logs -t grafana
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

# elasticsearch

es中一个index的字段类型如果在数据进入后，就被确认下来，不能更改。如果需要@timestamp为date类型，一定要在添加数据到es前操作。在创建index后，在mapping选项配置中，添加@timestamp为date类型，然后再推流数据。

## elasticsearch

```shell
sudo docker pull docker.elastic.co/elasticsearch/elasticsearch:8.17.2
sudo sysctl -w vm.max_map_count=262144
sudo docker run --name es01 --net elastic -p 9200:9200 -it -m 4GB docker.elastic.co/elasticsearch/elasticsearch:8.17.2
sudo docker start -i es01
```

内存需要配置为4GB否则会出现ERROR: Elasticsearch exited unexpectedly, with exit code 137

### 重置密码

记下重置后的密码

```shell
sudo docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

一般这个密码导出到环境变量

```
export ELASTIC_PASSWORD=
```



### 重置kibana连接token

记下token.

```shell
sudo docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

### 备份ca证书

```shell
sudo docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt .
```

elasticsearch是用自签名的ca证书，向外提供https服务。

### 测试是否安装正常

```shell
curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200
```

### 使用api key

先确认这个key有访问my-index的权限。

```shell
curl -k -H 'Authorization: ApiKey ******'  -X GET "https://localhost:9200/my-index/_mapping?pretty"
```

没有限制时会提示

```
elasticsearch.AuthorizationException: AuthorizationException(403, 'security_exception', 'action [cluster:monitor/main] is unauthorized for API key id [dyLlOpUBcNADYlky6Egd] of user [elastic], this action is granted by the cluster privileges [monitor,manage,all]')
```

这个表示没有权限monitor,manage,all

### python测试程序

```python
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

from elasticsearch import Elasticsearch

client = Elasticsearch(
  "https://localhost:9200",
  ca_certs="../../http_ca.crt",
  api_key="******",
)

print(client.info())

```

api_key在es中添加。client.info()权限是：

```json
    "cluster": [
      "monitor"
    ],
```

### 增加用户

首先要分配role

```
POST /_security/role/<role-name>
{
  "description": "Grants full access to all management features within the cluster.",
  "cluster": ["all"],
  "indices": [
    {
      "names": [ "*" ],
      "privileges": ["all"]
    }
  ]
}
```

然后让这个用户拥有这个role

```
POST /_security/user/<user-name>
{
  "password" : "<user-password>",
  "roles" : [ "<role-name>", "kibana_admin" ],
  "full_name" : "this is fullname"
}
```

kibana_admin表示这个用户可以登录管理kibana

最后测试一下

```
curl --cacert http_ca.crt -u <user-name>:<user-password> https://localhost:9200/my-index/_mapping?pretty
```



## kibana

### kibana

```shell
sudo docker pull docker.elastic.co/kibana/kibana:8.17.2
sudo docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.17.2
```

运行后会提示你访问哪个地址。

在浏览器中打开这个地址。

### 导入数据

数据格式为ndjson, 对象中一定要有@timestamp且为整数，单位是毫秒。导入完成后，在discover界面中确认。

https://github.com/ndjson/ndjson-spec

https://www.elastic.co/guide/en/kibana/8.17/connect-to-elasticsearch.html#_add_sample_data

### 综合管理Stack Management

在这里你可以管理index, dataview.
