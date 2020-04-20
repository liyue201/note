### 使用docker-compose部署单机版elasticsearch+kibana

操作系统： centos7.4


打开防火墙
```
$ firewall-cmd --list-ports
$ firewall-cmd --permanent --add-port=9200/tcp
$ firewall-cmd --permanent --add-port=5601/tcp
$ firewall-cmd --reload
```

安装docker

```
$ yum install docker
$ service docker start
```

安装docker-compose
```
$ yum install -y epel-release
$ yum install -y python-pip
$ pip install docker-compose
```

安装es+kibana

在docker-compose.yaml文件的目录下执行
```
$ docker-compose up -d 
```

docker-compose.yaml文件的内容如下：

```
version: '2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.7.1
    container_name: elasticsearch
    hostname: elasticsearch
    privileged: true
    environment:
      - discovery.type=single-node
    volumes:
      - ./esdata:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet

  kibana:
    image: kibana:6.7.1
    container_name: kibana
    ports:
      - 5601:5601
    environment:
      - ELASTICSEARCH_URL=http://elasticsearch:9200
    networks:
      - esnet
networks:
  esnet:
  
```
