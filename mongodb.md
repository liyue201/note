## docker安装mongoDB

```
docker run --name fil-mongo \
-v /lotus/filepp/mongo:/etc/mongo \
-e MONGO_INITDB_ROOT_USERNAME=admin \
-e MONGO_INITDB_ROOT_PASSWORD=hwht26 \
-e MONGO_INITDB_DATABASE=filespp \
-p 6163:27017 \
-d mongo:4.4.3
```

```
docker exec -it fil-mongo /bin/bash
```

```
mongo 127.0.0.1:27017 -u admin -p hwht26 --authenticationDatabase 'admin'	

```
```
use filepp

db.createUser(
  {
    user: "filepp",
    pwd: "gag435",
    roles: [ { role: "readWrite", db: "filepp" }]
  }
)
```
