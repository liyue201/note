
## Docker

### 批量删除

```
docker rm $(docker ps -a | grep "Exited" | awk '{print $1}')

docker rmi $(docker images | grep "none" | awk '{print $3}')


docker ps -a | grep "Exited" | awk '{print $1}'| xargs docker rm

docker images | grep none | awk '{print $3}'| xargs docker rmi
```

### 自动重启

自动重启

```
docker run -d --restart=always mysql
 
docker container update --restart=always  mysql
```

查看容器启动次数
```
docker inspect -f "{{ .RestartCount }}"  容器
 ```
查看容器最后一次启动时间
 ```
docker inspect -f "{{ .State.StartedAt }}"  容器
```
