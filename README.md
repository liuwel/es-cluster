# 使用 docker 创建 Elasticsearch 7.x 集群

#### Elasticsearch 版本：7.6.0
服务节点 node0,node1 （可以自行添加更多节点）
本机IP地址：172.16.0.17
node0 绑定端口 9200,9300
node1 绑定端口 9201,9301

####目录结构
```shell
es-cluster
|-- README.md
|-- config
|   |-- node0
|   |   `-- elasticsearch.yml
|   `-- node1
|       `-- elasticsearch.yml
`-- docker-compose.yml
```

es-cluster/config/node0/elasticsearch.yml
node0 配置内容(7.x相对于6.x 配置项有改动)：
```yaml
cluster.name: es-cluster
node.name: node0
node.master: true
node.data: true
node.attr.rack: r1
bootstrap.memory_lock: true
http.port: 9200
network.host: 172.16.0.17
transport.tcp.port: 9300
discovery.seed_hosts: ["172.16.0.17:9301"]
cluster.initial_master_nodes: ["node0"]
gateway.recover_after_nodes: 1
```

es-cluster/config/node1/elasticsearch.yml
node1 配置内容 (于node0配置类似 仅有部分改动)
```yaml
cluster.name: es-cluster
node.name: node1
node.master: true
node.data: true
node.attr.rack: r1
bootstrap.memory_lock: true
http.port: 9201
network.host: 172.16.0.17
transport.tcp.port: 9301
discovery.seed_hosts: ["172.16.0.17:9300"]
cluster.initial_master_nodes: ["node0"]
gateway.recover_after_nodes: 1
```

docker-compose.yml 内容
```yaml
version: "3"
services:
    es-node0:
        image: elasticsearch:7.6.0
        container_name: es-node0
        environment:
            - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
        restart: always
        volumes:
            - ./config/node0/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
            - es-node0:/usr/share/elasticsearch/data:rw
            - es-logs0:/usr/share/elasticsearch/logs:rw
        network_mode: "host"
        ports:
            - 9200:9200
            - 9300:9300
        ulimits:
            memlock:
                soft: -1
                hard: -1
    es-node1:
        image: elasticsearch:7.6.0
        container_name: es-node1
        environment:
            - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
        restart: always
        volumes:
            - ./config/node1/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
            - es-node1:/usr/share/elasticsearch/data:rw
            - es-logs1:/usr/share/elasticsearch/logs:rw
        network_mode: "host"
        ports:
            - 9201:9201
            - 9301:9301
        ulimits:
            memlock:
                soft: -1
                hard: -1
volumes:
    es-node0:
    es-node1:
    es-logs0:
    es-logs1:
```
### 启动服务
```shell
➜  es-cluster git:(master) sudo docker-compose up -d
Creating es-node1 ... 
Creating es-node0 ... 
Creating es-node0 ... 
Creating es-node0 ... done

➜  es-cluster git:(master) curl http://172.16.0.17:9200/_cat/nodes\?v
ip          heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.16.0.17           20          65  30    0.98    0.65     0.39 dilm      -      node1
172.16.0.17           22          65  30    0.98    0.65     0.39 dilm      *      node0

➜  es-cluster git:(master) curl http://172.16.0.17:9200/_cluster/health\?pretty
```
```json
{
  "cluster_name" : "es-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 2,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```
