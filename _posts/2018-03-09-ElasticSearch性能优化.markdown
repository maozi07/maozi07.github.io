---
layout: post
---

### ES优化
- 查看node上内存使用情况

```
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


### 一些官方的Blog（对于初学者很有帮助）
> 1. [关于thread pool的详细解释](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html)
> 2. [节点状态的监控](https://www.elastic.co/guide/en/elasticsearch/guide/current/_monitoring_individual_nodes.html)
> 3. [Bulk Thread pool rejection状况解释](https://www.elastic.co/blog/why-am-i-seeing-bulk-rejections-in-my-elasticsearch-cluster)
> 4. [理解内存压力状况](https://www.elastic.co/blog/found-understanding-memory-pressure-indicator)
