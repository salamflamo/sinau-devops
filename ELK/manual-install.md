# First install manual

## Install openjdk
- install jdk `yum install java-1.8.0-openjdk`

## Install Elasticsearch
- https://www.elastic.co/guide/en/elasticsearch/reference/7.7/rpm.html
- create repo `/etc/yum.repos.d/`
```
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```
- install elasticsearch `sudo dnf install --enablerepo=elasticsearch elasticsearch`
- configure elastic `/etc/elastisearch/elasticsearch.yml`
    - configure to `cluster.initial_master_nodes: ["192.168.36.4"]` ip for a master node 
    - configure to `discovery.seed_hosts: ["0.0.0.0"]` All host
    - configure to `network.host: 192.168.36.4` node ip
    - configure to `http.port: 9200` elastic port
- start elasticsearch `systemctl enable elasticsearch --now`

## Install Kibana
- install kibana `sudo dnf install --enablerepo=elasticsearch kibana`
- configure kibana `/etc/kibana/kibana.yml`
    - configure to `server.port: 5601`
    - configure to `server.host: "192.168.36.4"`
    - configure to `elasticsearch.hosts: ["http://192.168.36.4:9200"]`
- start kibana `systemctl enable kibana --now`

## Install Logstash (optional)
- install logstash `sudo dnf install --enablerepo=elasticsearch logstash`
- configure logstash `/etc/logstash/logstash.yml`
    - 
- start logstash `systemctl enable logstash --now`

## Install Filebeat
