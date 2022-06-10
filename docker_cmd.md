
## Docker

### 批量删除

```
docker rm $(docker ps -a | grep "Exited" | awk '{print $1}')

docker rmi $(docker images | grep "none" | awk '{print $3}')


docker ps -a | grep "Exited" | awk '{print $1}'| xargs docker rm

docker images | grep none | awk '{print $3}'| xargs docker rmi
```
