---
layout: post
---
### ES基础知识
- 查看node上内存使用情况

```
#查看集群健康状态
GET _cat/health?v
#查看节点状况
GET _cat/nodes?v
#查看segment内存占用
GET _cat/nodes?v&h=name,port,sm
```

-  Index segment

```
#查看segment数
GET _cat/segments?v
#segment合并可有效降低ElasticSearch内存占用
POST /index_name/_forcemerge?max_num_segments=5
```

- 内存占用详细

```
curl 'http://localhost:9200/_cluster/stats?humane&pretty'
GET /_nodes/stats/indices?human
```

- Thread pool信息

```
GET /_nodes/stats/thread_pool
GET /_cat/thread_pool?v
GET _nodes/thread_pool
```

- Index相关

```
#查看index分片状况
GET _cat/shards?v&index=*myindex*&h=index,shared,prirep,stat,node
#force merge segment
POST /*2020.09*/_forcemerge?max_num_segments=1
#禁用刷新，一般进行大量数据倒入时关闭刷新，倒入完成后打开；已经不再更新的index也可以禁用
PUT myindex2020.05*/_settings
{"index" : {
       "refresh_interval" : -1
   }
}
#查看index恢复情况
GET myindex-2018.05_2/_recovery?human
#关闭index，关闭已经不使用的index可提高集群性能
POST /myindex2017/_close
#打开index，打开已经关闭的index，需要一定的时间
POST /myindex2017/_open
#可以使用rollover功能来设置一些不能index，符合一定的条件后生成新的index，配合alias使用；以获得合适大小的index
POST /myindex/_rollover?dry_run
{
  "conditions": {
    "max_age":   "90d",  #时间长度90天
    "max_size":  "10gb"  #数据量达到10GB
  }
}
```

- 集群设置

```
#设置不可以换分分片的节点，配置项可以是一个或多个节点
PUT /_cluster/settings
{
  "transient":{
    "cluster.routing.allocation.exclude._name": "null"
  }
}
#配置部分节点可以划分分片，除去配置中的节点，其他节点都不可以划分分片
PUT /_cluster/settings
{
  "transient":{
    "cluster.routing.allocation.include._name": "an8XFEI,e_9GdbL,CrBTwW6,M8bpmYn"
  }
}
#设置所有节点可以划分分片
PUT /_cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.enable" : "all"
    }
}
#重新分配shard，以均衡集群负载；集群节点数有变化后，易出现数据不均衡
POST _cluster/reroute
{
  "commands": [
    {
      "move": {
        "index": "myindex2018.10.18-1",
        "shard": 0,
        "from_node": "a1111",
        "to_node": "b1111"
      }
    }
  ]
}

#ES磁盘满之后所有index会默认设置为readonly，取消readonly如下
PUT */_settings
{
  "index" : {
       "blocks.read_only_allow_delete" : "false"
   }

}
```


### 一些官方的Blog（对于初学者很有帮助）
> 1. [关于thread pool的详细解释](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html)
> 2. [节点状态的监控](https://www.elastic.co/guide/en/elasticsearch/guide/current/_monitoring_individual_nodes.html)
> 3. [Bulk Thread pool rejection状况解释](https://www.elastic.co/blog/why-am-i-seeing-bulk-rejections-in-my-elasticsearch-cluster)
> 4. [理解内存压力状况](https://www.elastic.co/blog/found-understanding-memory-pressure-indicator)
