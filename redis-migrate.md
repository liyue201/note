## redis数据迁移

从一个库复制到另一个库
```
#!/bin/bash
src_ip=127.0.0.1
src_port=6379
src_db=1
src_pw='123456'

dest_ip=127.0.0.1
dest_port=6379
dest_db=2
dest_pw='123456'

redis-cli -h $src_ip -p $src_port  -a $src_pw  -n $src_db keys "*" | while read key
do
    redis-cli -h $src_ip -p $src_port -a $src_pw  -n $src_db --raw dump $key | perl -pe 'chomp if eof' | redis-cli -h $dest_ip -p $dest_port -a $dest_pw -n $dest_db -x restore $key 0
    echo "migrate key $key"
done

```
