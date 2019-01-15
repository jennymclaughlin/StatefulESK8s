This brief instruction shows how to deploy a Elasticsearch cluster on Kubernetes by using the `StatefulSet` and `StorageClass`

## Deploy

```shell
kubectl create -f gce-standard-sc.yml
kubectl create -f dev-namespace.yml
kubectl create -f elasticsearch.yml
kubectl create -f es-discovery-svc.yml
kubectl create -f es-lb-svc.yml
```
Kubernetes creates the pods for a `StatefulSet` one at a time, waiting for each to come up before starting the next, so it may take a few minutes for all pods to be provisioned.
These use a `volumeClaimTemplates` to provision persistent storage for each pod.

## Storage

The [`elasticsearch.yml`](elasticsearch.yml) file contains `volumeClaimTemplates` sections which request 20GB volume for each pod. This will fill up quickly under moderate to heavy load. Consider modifying the disk size to your needs.

## Test
```shell
curl 130.211.238.144:9200
```

Download [Kibana](https://www.elastic.co/downloads/past-releases/kibana-5-6-0) from  and change the configuration. config/kibana.yml file
```shell
elasticsearch.url: "http://130.211.238.144:9200"
```

Go to Kibana directory and start Kibana
```shell
./bin/kibana
```
Load some data into Kibana by going to Dev Tools:
```shell
POST _bulk
{ "index" : { "_index" : "logs-2018-07-17", "_type" : "doc", "_id" : "4" } }
{"content":"Retrying individual bulk actions that failed","date":"2018-07-17T15:47:44,395","status":404,"shards":"1,2,3,4","retried_on_conflict":4,"roles":["superuser","ingest_admin"]}
{ "index" : { "_index" : "logs-2018-07-17", "_type" : "doc", "_id" : "3" } }
{"content":"failed to execute bulk item (index) BulkShardRequest","date":"2018-07-17T19:57:34,511","status":403,"shards":"1","retried_on_conflict":2,"roles":["monitoring_user"]}
{ "index" : { "_index" : "logs-2018-07-17", "_type" : "doc", "_id" : "5" } }
{"content":"Attempting to install template","date":"2018-07-17T20:05:01,678","status":200,"shards":"1,2","retried_on_conflict":5,"roles":["kibana_user","logstash_system"]}
{ "index" : { "_index" : "logs-2018-07-17", "_type" : "doc", "_id" : "12" } }
{"content":"creating index, cause [auto(bulk api)], templates [template_1], shards [1]/[0]","date":"2018-07-18T20:12:10,198","retried_on_conflict":1,"status":202,"shards":"0,1","roles":["kibana_user","ingest_admin"]}
{ "index" : { "_index" : "logs-2018-07-17", "_type" : "doc", "_id" : "14" } }
{"content":"Retrying individual bulk actions that failed","date":"2018-07-18T18:11:35,136","status":404,"shards":"1,2","retried_on_conflict":4,"roles":["kibana_user"]}
```
Search data
```shell
GET  logs-2018-07-17/_search
```
