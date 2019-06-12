## jenkins中使用ansible的坑

##### 问题：

已经在jenkins主机上配置到其他主机的ssh免密登录，执行ansible也正常，但是在jenkins中执行失败。

```
fatal: [192.168.101.231]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: Host key verification failed.", "unreachable": true}
```

##### 原因：
因为jenkins是以jenkins用户执行ansible的，通过ssh连接到其他机器时权限验证失败。

##### 解决办法：

```
vi /etc/ansible/ansible.cfg
```

```
# uncomment this to disable SSH key host checking
host_key_checking = False
```
