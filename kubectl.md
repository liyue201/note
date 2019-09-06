 ### kubectl实用命令


对于某些不方便手写的yaml文件，可以用--dry-run生成  
比如从二进制为文件生成configmap


```
 kubectl create configmap config --from-file=license.bin  -o yaml --dry-run >> conf.yaml
```
