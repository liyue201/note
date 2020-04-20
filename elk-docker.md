## 使用docker-compose部署单机版elasticsearch+kibana

操作系统： centos7.4


### 关闭防火墙
```
$ firewall-cmd --list-ports
$ firewall-cmd --permanent --add-port=9200/tcp
$ firewall-cmd --permanent --add-port=5601/tcp
$ firewall-cmd --reload
```

### 关闭SELinux
查看SELinux状态：

 ```
getenforce
```
```
Disabled
```
关闭SELinux：

临时关闭（不用重启机器）：

```
setenforce 0    ##设置SELinux 成为permissive模式
##setenforce 1 设置SELinux 成为enforcing模式
 ```

修改配置文件需要重启机器：

修改/etc/selinux/config 文件

将SELINUX=enforcing改为SELINUX=disabled

重启机器即可


### 安装docker

```
$ yum install docker
$ service docker start
```

### 安装docker-compose
```
$ yum install -y epel-release
$ yum install -y python-pip
$ pip install docker-compose
```

### 安装es+kibana

新建es数据挂载目录esdata，并修改权限
```
mkdir esdata
chomd 777 esdata
```

新建文件docker-compose.yaml, 文件的内容如下：

```
version: '2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.6.2
    container_name: elasticsearch
    hostname: elasticsearch
    privileged: true
    user: "1000:0"
    environment:
      - discovery.type=single-node
    volumes:
      - ./esdata:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet

  kibana:
    image: kibana:7.6.2
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

在docker-compose.yaml文件的目录下执行
```
$ docker-compose up -d 
```
