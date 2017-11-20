# ansible-beats-playbook

This playbook is for setting up version 6.x of Beats for shipping data to a remote Elasticsearch server:

- Metricbeat takes the metrics and statistics that it collects and ships them to the output that you specify, such as Elasticsearch or Logstash;

- Filebeat monitors the log directories or specific log files, tails the files, and forwards them either to Elasticsearch or Logstash for indexing.

# how to use

```bash
pip install ansible
mkdir -p /etc/ansible  
cat > /etc/ansible/hosts << EOT  
[beathosts]
192.168.1.111 ansible_user=centos  
EOT  
git clone https://github.com/considerable/ansible-beats-playbook.git  
cd ansible-beats-playbook  
sudo ansible-playbook site.yml 
```

# if need a local ELK for testing, use Docker images

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
