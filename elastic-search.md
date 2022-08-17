### \# Migrate Elasticsearch to another Elasticsearch
- https://alibaba-cloud.medium.com/how-to-migrate-elasticsearch-data-using-logstash-a1562184d995

### \# Tips and Tricks
- https://logz.io/blog/elasticsearch-cheat-sheet
- https://github.com/elastic/elasticsearch/issues/6409
- https://www.datadoghq.com/blog/elasticsearch-unassigned-shards/
- https://www.loggly.com/blog/nine-tips-configuring-elasticsearch-for-high-performance/
- https://kb.objectrocket.com/elasticsearch/how-to-perform-a-full-cluster-restart-for-elasticsearch-upgrade-158
- https://www.elastic.co/guide/en/elasticsearch/reference/6.8/shards-allocation.html
- https://gist.github.com/evert0n/3f5d1dce2ae107f17faf9318a5058400
- https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html

`Warning:` if you do not specify any action in curl command -X (PUT|POST|DELETE|GET), by default is -X GET

#### \# Main Variable
```bash
$ export ELKIP=xxx.xxx.xxx.xxx\
$ export ELKIP=${ELKIP:=127.0.0.1}
$ export ELKPORT=${ELKPORT:=9200}
```
#### \# GET Node nodename
```bash
$ curl -s  http://${ELKIP}:${ELKPORT}/_nodes\?pretty  | head -n10  | grep -w name | sed 's# ##g' | awk -F ':' '{print $2}' | sed 's#"##g ;  s#,##g'
```

#### \# GET Index
```bash
$ curl -s http://${ELKIP}:${ELKPORT}/_cluster/health?level=indices
```
#### \# GET Nodes Health
```bash
$ curl -s http://${ELKIP}:${ELKPORT}/_nodes?pretty 
```

## Hands-ON

#### \# Pending Task
```bash
$ curl -s http://${ELKIP}:${ELKPORT}/_cluster/pending_tasks?pretty
```

#### \# Health Check
```bash
$ curl -s http://${ELKIP}:${ELKPORT}/_cluster/health?pretty
```

#### \# Process
```bash
$ curl -s http://${ELKIP}:${ELKPORT}/_nodes/stats/process?pretty
```

#### \# Settings
```bash
$ curl -s "http://${ELKIP}:${ELKPORT}/_cluster/settings?pretty" -H 'Content-Type: application/json' 
```

#### \# Explain why Shards is unassigned
```bash
$ curl -s http://${ELKIP}:${ELKPORT}/_cluster/allocation/explain?pretty
```

#### \# Reroute Shards with status failed
```bash
$ curl -s -XPOST http://${ELKIP}:${ELKPORT}/_cluster/reroute?retry_failed=true
```

#### \# Show Shards without Assigned
```bash
$ curl -s "http://${ELKIP}:${ELKPORT}/_cat/shards?h=index,shard,prirep,state,unassigned.reason"| grep UNASSIGNED
```

#### \# Other infos
```bash
$ curl -s http://${ELKIP}:${ELKPORT}/_recovery?pretty

$ curl -s http://${ELKIP}:${ELKPORT}/_cat/thread_pool?v
```

#### \# Shards Reallocate
```bash
$ curl -s -X PUT http://${ELKIP}:${ELKPORT}/_cluster/settings -H 'Content-Type: application/json' -d '{"transient":{"cluster.routing.allocation.enable": "all"
    }
 }' 
```

#### \# Put read-only do index
```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_settings"  -H 'Content-Type: application/json' -d'
    {
    "index": {
    "blocks": {
    "read_only_allow_delete": "true" }
    }
}' 

$ curl -X PUT "http://${ELKIP}:${ELKPORT}/_settings" -H 'Content-Type: application/json' -d'
{ 
 "index.blocks.write": false 
}'
```

#### \# Migration assistance
```bash
$ curl -s  "http://${ELKIP}:${ELKPORT}/_xpack/migration/assistance"

$ curl -s -X POST "http://${ELKIP}:${ELKPORT}/_xpack/migration/upgrade/.watches

$ curl -s -X POST "http://${ELKIP}:${ELKPORT}/_xpack/migration/upgrade/.triggered_watches"

$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_settings" -H 'Content-Type: application/json' -d'
{
    "index" : {
        "refresh_interval" : "-1"
    }
}'
```

#### \# Watermark Adjust
```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_cluster/settings" -H 'Content-Type: application/json' -d'
{
  "transient": {
    "cluster.routing.allocation.disk.watermark.low": "90%",
    "cluster.routing.allocation.disk.watermark.high": "98%",
    "cluster.info.update.interval": "1m"
  }
}'


$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_all/_settings" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "0"
  }}'

```

#### \# Trying to recovery shards
```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_cluster/settings" -H 'Content-Type: application/json' -d '{
    "persistent" : {
   "cluster.routing.allocation.node_concurrent_recoveries" : 10
    }
}'
```

#### \# Adjust Delay timeout left
```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_all/_settings" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "10m"
  }
}'
```

#### \# Allocation enable for all shards
```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_all/_settings" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "index.routing.allocation.enable": "all"
 }
}'
```

#### \# Exclude one or more IP(s)
```bash
$ curl -s -X PUT http://${ELKIP}:${ELKPORT}/_cluster/settings -H 'Content-Type: application/json' -d '{
  "transient" :{
      "cluster.routing.allocation.exclude._ip" : "192.168.1.1,192.168.1.2"
   }
}'
```

#### \# By default replicas = 1 always, this setting change this number to 0, avoiding wasted space

```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_settings" -H 'Content-Type: application/json'  -d '
{
  "index" : {
    "number_of_replicas" : 0
  }
}'
```

#### \# Index bulk refresh
```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_settings" -H 'Content-Type: application/json'   -d '{
    "index" : {
        "refresh_interval" : "-1"
    }
}'


$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_settings" -H 'Content-Type: application/json'   -d '{
    "index" : {
        "refresh_interval" : "1s"
    }
}'
```

#### \# Fast Reindex
```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_settings" -H 'Content-Type: application/json'   -d '{
    "persistent" : {
        "indices.store.throttle.max_bytes_per_sec" : "100mb" 
    }
}'


$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_settings" -H 'Content-Type: application/json'  -d '{ 
    "index.merge.scheduler.max_thread_count" : 1
}'
```

#### \# Stats indices (size, date)
```bash
$ curl -XGET -s http://${ELKIP}:${ELKPORT}/_cat/indices?h=h,s,i,id,p,r,dc,dd,ss,creation.date.string

$ curl -XGET -s http://${ELKIP}:${ELKPORT}/_cat/indices?h=h,s,i,id,p,r,dc,dd,ss,creation.date.string
```

#### \# Reroute shards to another node
```bash
$ for index in $(curl -s -XGET "http://${ELKIP}:${ELKPORT}/_cat/shards?h=index,state" | grep UNASSIGNED | awk '{print $1}');
    do 
        for shard in $(curl -s -XGET "http://${ELKIP}:${ELKPORT}/_cat/shards?h=index,state,shard" | grep $index | grep UNASSIGNED | awk '{print $3}'); 
            do 
                curl -XPOST "http://${ELKIP}:${ELKPORT}/_cluster/reroute" -H 'Content-Type: application/json' -d '{
                    "commands": [{
                    "allocate": {
                    "index": "'${index}'",
                    "shard": '${shard}',
                    "node": "NAME-OF-NODE",
                    "allow_primary": 1
                             }
                        }
                    ] 
                 }';
        done; 
done
```

```bash
$ for line in $(curl -s http://${ELKIP}:${ELKPORT}/_cat/shards | grep UNASSIGNED | awk '{printf "%s;%s;%s\n", $1, $2, $8}' );  do
    IFS=';' read index shard node <<< "$line"
    curl -XPOST "http://${ELKIP}:${ELKPORT}/_cluster/reroute" -d '{
        "commands" : [ {
              "allocate" : {
                  "index" : "'$index'", 
                  "shard" : '$shard', 
                  "node" : "'$node'", 
                  "allow_primary" : true
              }
            }
        ]
    }'
done
```

#### \# NEW - Replica 0 for each Shard UNASSIGNED - most used 
```bash
 $ for index in $(curl -s "http://${ELKIP}:${ELKPORT}/_cat/shards?h=index,shard,prirep,state,unassigned.reason" | grep -i --color UNASSIGNED  | awk '{print $1}'); 
    do  
        curl -X PUT "http://${ELKIP}:${ELKPORT}/$index/_settings"  -H 'Content-Type: application/json' -d'
        {                
            "index" : {
                "number_of_replicas" : 0 
            }
        }'
done
```

#### \# Timeouts
```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
{
    "persistent" : {
        "timeout" : "120s"
    }
}'
```

```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
{
    "persistent" : {
        "master_timeout" : "120s"
    }
}'
```

#### \# Recovery expert

[+] info: https://www.elastic.co/guide/en/elasticsearch/reference/current/recovery.html

```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
{
    "transient" : {
        "indices.recovery.max_concurrent_file_chunks" : "5"
    }
}'

```

```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
{
    "transient" : {
        "indices.recovery.max_concurrent_operations" : "5"
    }
}'
```

```bash
$ curl -s  -X PUT "http://${ELKIP}:${ELKPORT}/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
{
    "transient" : {
        "indices.recovery.max_bytes_per_sec" : "500mb"
    }
}'
```

```bash
$ curl -s -X PUT "http://${ELKIP}:${ELKPORT}/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
{
    "transient" : {
        "cluster.routing.allocation.enable" : "all"
    }
}'
```
