docker安装clickhouse

```
docker run -d \
--name clickhouse \
--ulimit nofile=262144:262144 \
-p 9000:9000 \
-p 8123:8123 \
-e CLICKHOUSE_USER=root \
-e CLICKHOUSE_PASSWORD=123456 \
-e CLICKHOUSE_DB=test \
--volume=/data/clickhouse:/var/lib/clickhouse \
yandex/clickhouse-server:20.12
```
