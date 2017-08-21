## k8s集群监控cAdvisor+InfluxDB+Grafana

### 主机
* host1: 172.13.31.200
* host2: 172.13.31.163

其中host1 是k8s的工作节点，host2是监控数据的保存节点。


### 安装InfluxDB
在host2中安装InfluxDB，暴露端口18083和18086
```
docker run -d -p 18083:8083 -p 18086:8086 -v /var/influxdb:/data --name influxdb tutum/influxdb
```

打开InfluxDB的管理页面 ![http://host2:18083](http://host2:18083])，创建账test和密码test123，创建数据库cadvisor


### 安装cAdvisor
在host1安装cAdvisor，暴露端口18516，配置好前面设置的InfluxDB的ip、端口、数据库名称等。
```
docker run --volume=/:/rootfs:ro \
--volume=/var/run:/var/run:rw \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker/:/var/lib/docker:ro \
--publish=18516:8080 \
--detach=true  \
--name=cadvisor \
google/cadvisor:latest \
-storage_driver=influxdb \
-storage_driver_db=cadvisor \
-storage_driver_host=172.13.31.163:18086 
```
打开cAdvisor的页面 ![http://host1:18516](http://host1:18516])，可以看到服务器运行状况。


### 安装Grafana
在host2安装Grafana，暴露端口15892，配置好前面设置的InfluxDB的ip、端口、数据库名称、用户、密码等。
```
docker run -d -p 15892:3000 \
-e INFLUXDB_HOST=172.13.31.163 \
-e INFLUXDB_PORT=18086 \
-e INFLUXDB_NAME=cadvisor \
-e INFLUXDB_USER=test \
-e INFLUXDB_PASS=test123 \
--name grafana grafana/grafana
```
打开Grafana的页面 ![http://host2:15892](http://host2:15892])，通过admin/admin登录，设置数据源，创建dashboard等，参考网上的配置。
