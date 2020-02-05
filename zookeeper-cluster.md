### 使用docker部署zookeeper集群
操作系统：centos7.4

三台主机：
- host1： 10.102.20.124
- host2： 10.102.20.125
- host3： 10.102.20.126

关闭相关端口的防火墙
```
$ firewall-cmd --list-ports
$ firewall-cmd --permanent --add-port=2181/tcp
$ firewall-cmd --permanent --add-port=2888/tcp
$ firewall-cmd --permanent --add-port=3888/tcp
$ firewall-cmd --reload
```

安装docker
```
yum install docker
service docker start
```

每台机器上分别安装zk，唯一不同个是 ZOO_MY_ID

host1中执行
```
//host1

docker run \
--privileged=true \
--name zk \
--network=host \
-e "ZOO_INIT_LIMIT=10" \
-e "ZOO_MAX_CLIENT_CNXNS=500" \
-e "ZOO_MY_ID=1" \
-e "ZOO_SERVERS=server.1=10.102.20.124:2888:3888 server.2=10.102.20.125:2888:3888 server.3=10.102.20.126:2888:3888" \
-v /zk/data:/data \
-v /zk/log:/datalog \
-d zookeeper:3.4
```

host2中执行
```
docker run \
--privileged=true \
--name zk \
--network=host \
-e "ZOO_INIT_LIMIT=10" \
-e "ZOO_MAX_CLIENT_CNXNS=500" \
-e "ZOO_MY_ID=2" \
-e "ZOO_SERVERS=server.1=10.102.20.124:2888:3888 server.2=10.102.20.125:2888:3888 server.3=10.102.20.126:2888:3888" \
-v /zk/data:/data \
-v /zk/log:/datalog \
-d zookeeper:3.4
```

host3中执行
```
docker run \
--privileged=true \
--name zk \
--network=host \
-e "ZOO_INIT_LIMIT=10" \
-e "ZOO_MAX_CLIENT_CNXNS=500" \
-e "ZOO_MY_ID=3" \
-e "ZOO_SERVERS=server.1=10.102.20.124:2888:3888 server.2=10.102.20.125:2888:3888 server.3=10.102.20.126:2888:3888" \
-v /zk/data:/data \
-v /zk/log:/datalog \
-d zookeeper:3.4
```




