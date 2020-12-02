## 网络相关命令

##### 查看端口状态

```
lsof -i:8080
```
或
```
 netstat -tunlp | grep 8080
```

##### kill监听指定端口的进程

```
kill `lsof -i:8080 | grep LISTEN | awk -F " "  '{print $2}' `
```

## IO相关命令

查看iotop
```
iotop
```

