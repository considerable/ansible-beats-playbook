# ansible-beats-playbook

This playbook is for setting up version 6.x of Beats for shipping data to a remote Elasticsearch server:

- Metricbeat takes the metrics and statistics that it collects and ships them to the output that you specify, such as Elasticsearch or Logstash;

- Filebeat monitors the log directories or specific log files, tails the files, and forwards them either to Elasticsearch or Logstash for indexing.

## How to use: replace 192.168.1.111 with your IP

```bash
pip install ansible
mkdir -p /etc/ansible  
cat > /etc/ansible/hosts << EOT  
[beathosts]
192.168.1.111 ansible_user=centos  
EOT  
git clone https://github.com/considerable/ansible-beats-playbook.git  
cd ansible-beats-playbook  
ansible-playbook site.yml 
```

## For local ElasticSearch + Kibana testing, use Docker images

```bash
# ::::::::::::::
# docker-elasticsearch/docker-compose.yml
# ::::::::::::::
# per https://www.elastic.co/guide/en/elasticsearch/reference/6.0/docker.html
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.0.0
    container_name: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.0.0
    container_name: elasticsearch2
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata2:/usr/share/elasticsearch/data
    networks:
      - esnet
volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local
networks:
  esnet:

# :::::::::::::: 
# docker-kibana/docker-compose.yml
# ::::::::::::::
# per https://www.elastic.co/guide/en/kibana/6.0/_configuring_kibana_on_docker.html
version: '2'
services:
  kibana:
    image: docker.elastic.co/kibana/kibana:6.0.0
    ports:
      - 5601:5601
    environment:
      SERVER_NAME: '0.0.0.0'
      ELASTICSEARCH_URL: http://192.168.1.111:9200
```

## Testing query    

curl -s 'localhost:9200/_search?size=1&pretty&q=system.filesystem.type:rootfs'

```
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 6,
    "successful" : 6,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 79,
    "max_score" : 2.8291354,
    "hits" : [
      {
        "_index" : "metricbeat-6.0.0-2017.11.20",
        "_type" : "doc",
        "_id" : "kay4218Bqg2UCBXZVGB_",
        "_score" : 2.8291354,
        "_source" : {
          "@timestamp" : "2017-11-20T23:16:50.407Z",
          "metricset" : {
            "module" : "system",
            "name" : "filesystem",
            "rtt" : 45665
          },
          "system" : {
            "filesystem" : {
              "type" : "rootfs",
              "total" : 107321753600,
              "used" : {
                "pct" : 0.0634,
                "bytes" : 6799953920
              },
              "free" : 100521799680,
              "free_files" : 104678869,
              "device_name" : "rootfs",
              "mount_point" : "/",
              "files" : 104857600,
              "available" : 100521799680
            }
          },
          "beat" : {
            "name" : "localhost",
            "hostname" : "localhost",
            "version" : "6.0.0"
          }
        }
      }
    ]
  }
}
```  
