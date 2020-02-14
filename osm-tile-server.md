## 部署openstreetmap瓦片服务器


### 下载地图数据
到[这里](http://download.geofabrik.de/asia/)下载最新的中国地图数据 china-200213.osm.pbf


### 创建pg数据库volume
```
docker volume create openstreetmap-data
```

### 导入地图数据到pg

在地图数据的目录下执行
```
docker run \
--rm \
-v $PWD/china-200213.osm.pbf:/data.osm.pbf \
-v openstreetmap-data:/var/lib/postgresql/12/main \
overv/openstreetmap-tile-server:1.3.8 \
import
```

### 运行服务
```
docker run \
--name osm \
-p 8080:80 \
-v openstreetmap-data:/var/lib/postgresql/12/main \
-v $PWD/tiles:/var/lib/mod_tile \
-d overv/openstreetmap-tile-server:1.3.8 \
run
```
tiles目录是瓦片数据的缓存目录

### 验证
打开链接 http://127.0.0.1:8080


### 参考
https://github.com/Overv/openstreetmap-tile-server


### 服务组件和工具
- PostGIS： 地图数据库，。 http://download.osgeo.org/postgis/source/postgis-3.0.0.tar.gz
- osm2pgsql：OSM格式转PG sql工具。 https://github.com/openstreetmap/osm2pgsql.git
- mod_tile、renderd： 生成瓦片的服务 https://github.com/SomeoneElseOSM/mod_tile.git
- openstreetmap-carto： 瓦片的样式 https://github.com/gravitystorm/openstreetmap-carto.git 
- Apache2： web服务


### 将pg和瓦片服务器分开部署
没有试过，可以参考  
https://github.com/Overv/openstreetmap-tile-server/issues/21  
https://help.openstreetmap.org/questions/51521/mapnik-and-postgresql-in-differents-servers  

