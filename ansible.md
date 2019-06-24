## jenkins中使用ansible的坑

##### 问题：

已经在jenkins主机上配置到其他主机的ssh免密登录(注意是本地jenkins账号（不是root账号）到远程root账号的免密)，执行ansible也正常，但是在jenkins中执行失败。

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

同时指定远程服务器用root账号执行  
可以在hosts文件中修改

```
[servers]
192.168.8.23  ansible_ssh_user=root
```

或者在playbook中修改
```
---
- name: deploy
  hosts: 192.168.8.23
  remote_user: root
```

或者在ansible.cfg 全局修改
```
# default user to use for playbooks if user is not specified
# (/usr/bin/ansible will use current user as default)
#remote_user = root
```

