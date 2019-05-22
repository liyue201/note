## 3台主机实现3主3从redis集群部署
操作系统： centos 7.4

3台主机的ip
- host1： 10.102.20.135
- host2： 10.102.20.136
- host3： 10.102.20.137

3主3从架构，每台主机一主一从，同一组主从不在同一台机器上，形成循环

|master |  slave |
| --------  | -----  |
|host1:7000  |  host2:7001|
|host2:7000  |  host3:7001|
|host3:7000  |  host1:7001|


在host1中配置变量
```
$ host1=10.102.20.135
$ host2=10.102.20.136
$ host3=10.102.20.137
```

关闭防护墙
```
$ firewall-cmd --list-ports
$ firewall-cmd --permanent --add-port=7000/tcp && \
  firewall-cmd --permanent --add-port=7001/tcp && \
  firewall-cmd --permanent --add-port=17000/tcp && \
  firewall-cmd --permanent --add-port=17001/tcp && \
  firewall-cmd --reload
```

安装gcc
```
$ yum -y install gcc
$ yum -y install tcl
```

取代码编译
```
$ wget http://download.redis.io/releases/redis-3.2.12.tar.gz
$ tar -xf redis-3.2.12.tar.gz
$ cd redis-3.2.12
$ make
```

测试一下
```
$ make test
```
  
在/opt下创建redis目录，将可执行文件，配置文件，日志文件都放在/opt/redis目录下
```
mkdir  -pv /opt/redis/conf /opt/redis/bin /opt/redis/log  /opt/redis/data/7000  /opt/redis/data/7001
```

拷贝常用的二进制文件
```
$ cp ./src/redis-* /opt/redis/bin
```
 
删除.c，.o的文件
```
$ rm /opt/redis/bin/*.c  /opt/redis/bin/*.o
```

修改环境变量
```
$ vim /etc/profile  #添加以下内容

export REDIS_HOME=/opt/redis
export PATH=${REDIS_HOME}/bin:$PATH

$ source /etc/profile
```

在host1上配置redis的主，从两个配置文件，下面是主实例的配置文件内容。
只是端口和目录位置不一样
```
$ vim /opt/redis/conf/redis7000.conf

bind 10.102.20.135
port 7000
cluster-enabled  yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
timeout 3600
daemonize yes

pidfile /opt/redis/pid/redis7000.pid
logfile "/opt/redis/log/redis7000.log"
dir "/opt/redis/data/7000"
  
rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command KEYS ""
```

```
$ vim /opt/redis/conf/redis7001.conf

bind 10.102.20.135
port 7001
cluster-enabled  yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
timeout 3600
daemonize yes

pidfile /opt/redis/pid/redis7000.pid
logfile "/opt/redis/log/redis7000.log"
dir "/opt/redis/data/7000"
  
rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command KEYS ""
```

拷贝到其他两台机器上
```
$ scp -r /opt/redis ${host2}:/opt
$ scp -r /opt/redis ${host3}:/opt
```

在host2、host3中修改redis7000.conf、redis7000.conf中的ip
修改环境变量
```
$ vim /etc/profile  #添加以下内容

export REDIS_HOME=/opt/redis
export PATH=${REDIS_HOME}/bin:$PATH

$ source /etc/profile
```

  
启动redis实例,3台机器均要启动
```
$ redis-server /opt/redis/conf/redis7000.conf && redis-server /opt/redis/conf/redis7001.conf
```
安装集群
在host1安装redis for  ruby客户端，依赖ruby2.2.2以上版本，这里报错，需要升级ruby版本

```
$ yum -y install ruby rubygems
$ gem install redis 
```

安装ruby2.4.4
```
yum install curl -y
```

安装rvm
```
$ curl -L get.rvm.io | bash -s stable
```
若是报错请按，提示执行
```
   gpg2 --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
```

```
$ source /usr/local/rvm/scripts/rvm
$ rvm list known
$ rvm install 2.4.4
$ gem install redis 
```

创建3个master节点的集群

```
$ redis-trib.rb create ${host1}:7000 ${host2}:7000 ${host3}:7000 
  >>> Creating cluster
  >>> Performing hash slots allocation on 3 nodes...
  Using 3 masters:
  10.102.20.135:7000
  10.102.20.136:7000
  10.102.20.137:7000
  M: 050c87bbafb0ae38671cc6bd72dd782b59113938 10.102.20.135:7000
     slots:0-5460 (5461 slots) master
  M: e77ed380f73ab72bae8459a72a31191422c13b50 10.102.20.136:7000
     slots:5461-10922 (5462 slots) master
  M: 1ad034189f9d0052718cc8efec569242e634cef3 10.102.20.137:7000
     slots:10923-16383 (5461 slots) master
  Can I set the above configuration? (type 'yes' to accept): yes
  >>> Nodes configuration updated
  >>> Assign a different config epoch to each node
  >>> Sending CLUSTER MEET messages to join the cluster
  Waiting for the cluster to join.
  >>> Performing Cluster Check (using node 10.102.20.135:7000)
  M: 050c87bbafb0ae38671cc6bd72dd782b59113938 10.102.20.135:7000
     slots:0-5460 (5461 slots) master
     0 additional replica(s)
  M: 1ad034189f9d0052718cc8efec569242e634cef3 10.102.20.137:7000
     slots:10923-16383 (5461 slots) master
     0 additional replica(s)
  M: e77ed380f73ab72bae8459a72a31191422c13b50 10.102.20.136:7000
     slots:5461-10922 (5462 slots) master
     0 additional replica(s)
  [OK] All nodes agree about slots configuration.
  >>> Check for open slots...
  >>> Check slots coverage...
  [OK] All 16384 slots covered.

```

若是卡在waiting...
在host2 上

```
$ redis-cli -c -h 10.102.20.136 -p 7000
10.102.20.136:7000> cluster meet 10.102.20.135 7000
OK
10.102.20.136:7000> cluster meet 10.102.20.136 7000
OK
```
在host3 上
```
$ redis-cli -c -h 10.102.20.137 -p 7000
10.102.20.136:7000> cluster meet 10.102.20.135 7000
OK
10.102.20.136:7000> cluster meet 10.102.20.136 7000
OK
```

添加slave节点到集群,保证每个master的slave不在同一台主机上
```
$ redis-trib.rb add-node --slave --master-id ${master1_id} ${host2}:7001 ${host1}:7000
$ redis-trib.rb add-node --slave --master-id ${master2_id} ${host3}:7001 ${host1}:7000
$ redis-trib.rb add-node --slave --master-id ${master3_id} ${host1}:7001 ${host1}:7000
```
实际命令如下：
```
$ redis-trib.rb add-node --slave --master-id 050c87bbafb0ae38671cc6bd72dd782b59113938 ${host2}:7001 ${host1}:7000
$ redis-trib.rb add-node --slave --master-id e77ed380f73ab72bae8459a72a31191422c13b50 ${host3}:7001 ${host1}:7000
$ redis-trib.rb add-node --slave --master-id 1ad034189f9d0052718cc8efec569242e634cef3 ${host1}:7001 ${host1}:7000
```

检查集群状态
```
$ redis-trib.rb check ${host1}:7000
>>> Performing Cluster Check (using node 10.102.20.137:7000)
M: 1ad034189f9d0052718cc8efec569242e634cef3 10.102.20.137:7000
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: 050c87bbafb0ae38671cc6bd72dd782b59113938 10.102.20.135:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: e77ed380f73ab72bae8459a72a31191422c13b50 10.102.20.136:7000
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: e6d5bb3f0eb47cbd710457180cb30cb363352db4 10.102.20.135:7001
   slots: (0 slots) slave
   replicates 1ad034189f9d0052718cc8efec569242e634cef3
S: 021d556632812f2d565a8a327029a1ae28a09f97 10.102.20.136:7001
   slots: (0 slots) slave
   replicates 050c87bbafb0ae38671cc6bd72dd782b59113938
S: d1e5f342db407b065eaccda2c8e53e1305b43125 10.102.20.137:7001
   slots: (0 slots) slave
   replicates e77ed380f73ab72bae8459a72a31191422c13b50
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
