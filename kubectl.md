 ### kubectl实用命令


对于某些不方便手写的yaml文件，可以用--dry-run生成  
比如从二进制为文件生成configmap


```
 kubectl create configmap config --from-file=license.bin  -o yaml --dry-run >> conf.yaml
```

### k8s容器异常退出调试
有事容器启动时候就挂了，没法进容器中去看。可以使用shell执行死循环方式启动，在进容器启动应用。

```
command: ["/bin/sh"]
args: ["-c", "while true; do echo hello; sleep 10;done"]

```
