- 检查es集群状态
```shell
curl -XGET http://localhost:9200/_cluster/health\?pretty
```

- 单节点es副本分片设为0
```shell
curl -XPUT 'http://localhost:9200/_settings' -H 'content-Type:application/json' -d'
{
    "number_of_replicas": 0
}'
```

- 查看es所有索引
```shell
curl -XGET http://localhost:9200/_cat/indices
```

- 删除30天之前索引
```shell
date=`date -d "-30 days" "+%Y.%m.%d"`
echo $date
/usr/bin/curl -XDELETE "http://localhost:9200/*-$date*"
```