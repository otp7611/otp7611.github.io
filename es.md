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

