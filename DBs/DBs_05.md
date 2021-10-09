# 6.5. Elasticsearch

#### 1
```bash
vagrant@vagrant:~$ cat Dockerfile 
FROM    centos:7

RUN     yum update -y && \
        yum install wget \ 
        perl-Digest-SHA -y

RUN     wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.0-linux-x86_64.tar.gz && \       
        wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.0-linux-x86_64.tar.gz.sha512 && \
        shasum -a 512 -c elasticsearch-7.15.0-linux-x86_64.tar.gz.sha512 && \ 
        tar -xzf elasticsearch-7.15.0-linux-x86_64.tar.gz

ENV     JAVA_HOME=/elasticsearch-7.15.0/jdk
ENV     ES_HOME=/elasticsearch-7.15.0

COPY    ./elasticsearch.yml /elasticsearch-7.15.0/config

RUN     useradd -ms /bin/bash elasticsearch && \
        chown -R elasticsearch:elasticsearch /elasticsearch-7.15.0 && \
        chown elasticsearch:elasticsearch /var/lib

USER    elasticsearch 

WORKDIR elasticsearch-7.15.0

EXPOSE 9200 

CMD ["./bin/elasticsearch"]

{
  "name" : "netology_test",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "9QW0FwztRXKQyHG2oGpzhw",
  "version" : {
    "number" : "7.15.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "79d65f6e357953a5b3cbcc5e2c7c21073d89aa29",
    "build_date" : "2021-09-16T03:05:29.143308416Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
docker push miost/elasticsearch-netology-test:latest
```
#### 2
```bash
curl -X PUT localhost:9200/int-1 -H 'Content-Type:application/json' -d '{"settings" : {"number_of_shards": 1, "number_of_replicas" : 0}}'
{"acknowledged":true,"shards_acknowledged":true,"index":"int-1"}

curl -X PUT localhost:9200/int-2 -H 'Content-Type:application/json' -d '{"settings" : {"number_of_shards": 2, "number_of_replicas" : 1}}'
{"acknowledged":true,"shards_acknowledged":true,"index":"int-2"}

curl -X PUT localhost:9200/int-3 -H 'Content-Type:application/json' -d '{"settings" : {"number_of_shards": 4, "number_of_replicas" : 2}}'
{"acknowledged":true,"shards_acknowledged":true,"index":"int-3"}

curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases VTWokjnsRQ-5Wnb-M0X2SA   1   0         41            0     40.1mb         40.1mb
green  open   int-1            WQxN-bp0R-KcG0sDT6TeXQ   1   0          0            0       208b           208b
yellow open   int-3            ULU8x6CPQ8Cu-vcJQpSFdA   4   2          0            0       832b           832b
yellow open   int-2            jcDBlzFZRP6NHw1t_79WGw   2   1          0            0       416b           416b

curl -X GET 'http://localhost:9200/_cluster/health/int-1?pretty'
{
  "cluster_name" : "elasticsearch",
  "status" : "green",
  "timed_out" : false,        
  "number_of_nodes" : 1,      
  "number_of_data_nodes" : 1, 
  "active_primary_shards" : 1,
  "active_shards" : 1,    
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,  
  "active_shards_percent_as_number" : 100.0
}

curl -X GET 'http://localhost:9200/_cluster/health/int-2?pretty'
{
  "cluster_name" : "elasticsearch",
  "status" : "yellow",
  "timed_out" : false,        
  "number_of_nodes" : 1,      
  "number_of_data_nodes" : 1, 
  "active_primary_shards" : 2,
  "active_shards" : 2,        
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 2,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 44.44444444444444
}

curl -X GET 'http://localhost:9200/_cluster/health/int-3?pretty'
{
  "cluster_name" : "elasticsearch",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,      
  "number_of_data_nodes" : 1, 
  "active_primary_shards" : 4,
  "active_shards" : 4,
  "relocating_shards" : 0,        
  "initializing_shards" : 0,      
  "unassigned_shards" : 8,        
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,  
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 44.44444444444444
}

curl -X GET 'http://localhost:9200/_cluster/health?pretty'
{
  "cluster_name" : "elasticsearch",
  "status" : "yellow",        
  "timed_out" : false,        
  "number_of_nodes" : 1,      
  "number_of_data_nodes" : 1, 
  "active_primary_shards" : 8,
  "active_shards" : 8,
  "relocating_shards" : 0,        
  "initializing_shards" : 0,      
  "unassigned_shards" : 10,       
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,  
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 44.44444444444444
}

Думаю, статус YELLOW из-за того, что все "реплики" находятся на одном сервере и по факту, не являются репликами.

curl -X DELETE 'http://localhost:9200/int-1?pretty'
{
  "acknowledged" : true
}

curl -X DELETE 'http://localhost:9200/int-2?pretty'
{
  "acknowledged" : true
}

curl -X DELETE 'http://localhost:9200/int-3?pretty'
{
  "acknowledged" : true
}
```
#### 3
```bash 
curl -X PUT 'localhost:9200/_snapshot/netology_backup' -H 'Content-type: application/json' -d '{"type": "fs", "settings": {"location": "/elasticsearch-7.15.0/snapshots"}}'
{"acknowledged":true}

curl -X PUT 'localhost:9200/test' -H 'Content-type: application/json' -d '{"settings": {"number_of_shards": 1, "number_of_replicas": 0}}'
{"acknowledged":true,"shards_acknowledged":true,"index":"test"}

curl -X GET 'localhost:9200/_cat/indices?pretty'
green open .geoip_databases Hc5rtw3vTcOqEn4YR4ut6Q 1 0 41 0 40.1mb 40.1mb
green open test             lZfZErkSR9GRdBI_lgxHIg 1 0  0 0   208b   208b

ls snapshots/
index-2  index.latest  indices  meta-cIa9Ykn8THi5oueBoIX-DQ.dat  snap-cIa9Ykn8THi5oueBoIX-DQ.dat

curl -X DELETE 'localhost:9200/test?pretty'
{
  "acknowledged" : true
}

curl -X PUT 'localhost:9200/test-2?pretty'
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test-2"
}

curl -X GET 'localhost:9200/_cat/indices?pretty'
green  open .geoip_databases Hc5rtw3vTcOqEn4YR4ut6Q 1 0 41 0 40.1mb 40.1mb
yellow open test-2           tSCNXmP5Ttyz15Vl8H95-A 1 1  0 0   208b   208b

curl -X POST 'localhost:9200/_snapshot/netology_backup/snapshot/_restore' -H 'Content-type: application/json' -d '{"indices": "test"}'
{"accepted":true}

curl -X GET 'localhost:9200/_cat/indices?pretty'
green  open .geoip_databases Hc5rtw3vTcOqEn4YR4ut6Q 1 0 41 0 40.1mb 40.1mb
yellow open test-2           tSCNXmP5Ttyz15Vl8H95-A 1 1  0 0   208b   208b
green  open test             YrumWluDRDeFT3QHjNOT_g 1 0  0 0   208b   208b

```
