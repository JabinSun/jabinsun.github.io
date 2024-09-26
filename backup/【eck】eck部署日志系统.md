> Kubernetes 1.26-1.30（ECK2.14.0，低版本可以支持低版本的K8S，请自行到官网查看。）
> Elasticsearch, Kibana, APM Server: 6.8+, 7.1+, 8+
> Beats: 7.0+, 8+
> Logstash: 8.7+

- operator
```shell
kubectl create -f https://download.elastic.co/downloads/eck/2.14.0/crds.yaml
kubectl apply -f https://download.elastic.co/downloads/eck/2.14.0/operator.yaml
```

- es
> 7.17.24
> 3master节点，master，data共用节点。
> HTTP模式
> basic认证
> hostNetwork
> local-path
```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: eslogs
  namespace: elastic-system
spec:
  version: 7.17.24
  http:
    tls:
      selfSignedCertificate:
        disabled: true
  nodeSets:
  - name: data
    count: 3
    podTemplate:
      spec:
        initContainers:
        - name: sysctl
          securityContext:
            privileged: true
            runAsUser: 0
          command: ['sh', '-c', 'sysctl -w vm.max_map_count=262144']
        containers:
        - name: elasticsearch
          env:
          - name: ES_JAVA_OPTS
            value: "-Xms16g -Xmx16g"
          resources:
            limits:
              cpu: 8
              memory: 32Gi
            requests:
              cpu: 500m
              memory: 512Mi
          volumeMounts:
          - name: timezone-volume
            mountPath: /etc/localtime
            readOnly: true
        volumes:
        - name: timezone-volume
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
        affinity:
          podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  elasticsearch.k8s.elastic.co/cluster-name: eslogs
              topologyKey: kubernetes.io/hostname
        hostNetwork: true
        dnsPolicy: ClusterFirstWithHostNet
    volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 3000Gi
        storageClassName: local-path
```

- kibana
```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: kibana
  namespace: elastic-system
spec:
  version: 7.17.24
  count: 3
  elasticsearchRef:
    # 对应上面ES资源的名称和命名空间
    name: eslogs
    namespace: elastic-system
  http:
    service:
      spec:
        type: NodePort
    tls:
      selfSignedCertificate:
        disabled: true
  podTemplate:
    spec:
      containers:
      - name: kibana
        env:
        - name: NODE_OPTIONS
          value: "--max-old-space-size=2048"
        - name: I18N_LOCALE
          value: zh-CN
        - name: SERVER_PUBLICBASEURL
          value: "http://log.xxxxxx.com"
        resources:
          requests:
            memory: 100Mi
            cpu: 0.5
          limits:
            memory: 4Gi
            cpu: 2
        volumeMounts:
        - name: timezone-volume
          mountPath: /etc/localtime
          readOnly: true
      volumes:
      - name: timezone-volume
        hostPath:
          path: /usr/share/zoneinfo/Asia/Shanghai
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                elasticsearch.k8s.elastic.co/name: kibana
            topologyKey: kubernetes.io/hostname
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
```
> # 解析: Secret: eslogs-es-elastic-user 获取es密码

- es数据迁移
> 迁移方案: reindex
> 设置索引模板，保证写入速率最大化
```shell
# 完成迁移后在索引管理中将number_of_replicas和refresh_interval参数设置为合适的值即可。
PUT _template/hwprod
{
  "index_patterns": [
    "hwprod*"
  ],
  "order": 999,
  "settings": {
    "refresh_interval": "-1",
    "number_of_shards": "3",
    "translog": {
      "sync_interval": "60s",
      "durability": "async"
    },
    "number_of_replicas": "0"
  }
}
```
> 配置集群白名单，新增源数据集群es地址
```shell
# 修改配置文件方式
vim /etc/elasticsearch/elasticsearch.yml 
reindex.remote.whitelist: "10.1.2.3:9200"
systemctl restart elasticsearch.service 

# K8S部署增加环境变量方式
# 修改自定义资源的ES资源，增加环境变量
- name: ES_SETTING_REINDEX_REMOTE_WHITELIST
  value: 10.1.2.3:9200
```
> 执行reindex迁移
```shell
# 请求
# size: 每批次的数据量
# slice: 切片数量, 和节点数一致
# wait_for_completion=false: 异步执行
POST _reindex?wait_for_completion=false
{
  "source": {
    "remote": {
      "host": "http://10.1.2.3:9200",
      "username": "elastic",
      "password": "asdf1234"
    },
    "index": "hwprod-2024.9.1",
    "size": 10000,
    "slice": {
      "id": 0,
      "max": 3
    }
  },
  "dest": {
    "index": "hwprod-2024.9.1"
  }
}

# 响应
{
  "task": "PuSABDEeTsSXUiW7U--uAw:40193"
}

# 查询异步任务
GET _tasks/PuSABDEeTsSXUiW7U--uAw:40193
```