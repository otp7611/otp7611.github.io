elasticsearch的使用

# 使用data streams

data streams流按日期使用多个indices保存数据。通过配置[index lifecycle management (ILM)](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html) 来管理这些indices，比如超过某个条件后自动删除旧的indices. data streams对外层来讲就是一个普通的index. 所以如果你想在kibana中查看数据，需要对这个index创建data view.

data streams在使用上的原则是只追加数据，不会更新中间数据（删除，改写之类的操作）。

## 创建data streams

### 创建index lifecycle policy

my-lifecycle-policy  配置indice的生存期。注意，只要有一个index使用了这个policy，这个policy就不能被改动。

### 创建component templates

my-mappings  注意，和index一样data streams在使用前一定要配置关键映射，而一定要配置@timestamp

my-lifecycle-policy  注意，这个选中刚才配置my-lifecycle-policy

```
"template": {
    "settings": {
      "index.lifecycle.name": "my-lifecycle-policy"
    }
}
```

### 创建index template

my-index-template使用刚才两个组件模板。

```console
"composed_of": [ "my-mappings", "my-settings" ],
```

"index_patterns": ["my-data-stream*"], index匹配时，对应的data streams会自动创建。

## 使用python向data streams中推数据

[Indexing requests](https://www.elastic.co/guide/en/elasticsearch/reference/current/use-a-data-stream.html#add-documents-to-a-data-stream) add documents to a data stream. These requests must use an `op_type` of `create`. Documents must include a `@timestamp` field.

向data streams中推数据时，op_type一定要是create，且要有@timestamp。

```
             records.append({"create": { "_index": indexName}})
             records.append(item)
```

如果不是data streams那么是：

```
             records.append({"index": { "_index": indexName}})
             records.append(item)
```

# 使用Time series data stream (TSDS)

在ES中如果字段是"A.B":???，那么等价于"A": {"B": ???}

Time series data stream是data stream的子类，data  stream的约束在Time series data stream都存在。

TSDS还有如下特殊的地方：

## _id

the document `_id` is a hash of the document’s dimensions and `@timestamp`.也就是说，_id是唯一的，这个唯一来自(dimensions, @timestamp)的组合唯一。如果不唯一，则入库失败。

## index.mode

{  "index": {  "number_of_replicas": "0",   "lifecycle": {      "name": "testtsds-lifecycle-policy"    },    "mode": "time_series"  } } 就是index.mode必须是time_series

## index.routing_path

必须指定哪些字段是dimensions，不能为空。这个dimension字段可以在mapping中用"time_series_dimension": true来标记。比如：

```
{
  "dynamic_templates": [],
  "properties": {
    "testid": {
      "time_series_dimension": true,
      "type": "keyword"
    },
    "testclass": {
      "time_series_dimension": true,
      "type": "keyword"
    }
  }
}
```

# api_key权限



```
{
  "my-index-key-role": {
    "cluster": [
      "monitor"
    ],
    "indices": [
      {
        "names": [
          "testtsdsindex",          
          "my-index"
        ],
        "privileges": [
          "auto_configure",
          "create_index",
          "manage",
          "all"
        ],
        "allow_restricted_indices": false
      }
    ],
    "applications": [],
    "run_as": [],
    "metadata": {},
    "transient_metadata": {
      "enabled": true
    }
  }
}
```

ES的权限是由role作组合的。一个用户需要权限的话，就要去选择这些role. api_key的权限设置时直接定义了一个role，在这里就是my-index-key-role。

# index lifecycle management (ILM)

如果indice的健康状态不是green，它是不会进入delete阶段的。

如果是单节点的es, 但是你没有配置number_of_replicas，那么它会默认为1，且始终为yellow导致无法被删除。所以需要把它显示配置为0.

配置为0后，它就自动会变成了green. 

进入delete阶段，它不一定会立即删除，但是它最终会被删除。

## ILM日志

ILM的日志也是一个data stream, 名字是ilm-history-7,这种ES内部的data stream. 在这里可以查看ES的ILM管理的所有indice. 包括indice进入到什么状态，什么时候进入此状态的等等。

# 查询数据

```
GET /vtsdsindex/_search
{
  "size": 50,
  "query": {
    "bool": { 
      "must": [
        { 
          "match": { 
            "source_id": "this is id" 
          }
        }
      ],
      "filter": [
        {
          "range": {
            "@timestamp": {
              "gte": "now-5h",
              "lte": "now",
              "format": "strict_date_optional_time_nanos"
            }
          }
        }
      ]
    }    
  }
}
```

# 强制datastream产生新的indice

```
POST /<your_alias_name>/_rollover
```

