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



***

## 部署openmaptiles瓦片服务器

openmaptiles是[Maptiler](https://openmaptiles.com/)公司开发的瓦片服务器，没有开源，但有免费的版本可以使用。openmaptiles比openstreetmap样式更多，而且美观，但是免费版的上面有很多水印。

使用官方的docker镜像部署，外在当前目录作为文件目录。

```
docker run --name osm -d -v $PWD:/data -p 8080:80 klokantech/openmaptiles-server
```

启动容器后，打开网页 http://127.0.0.1:8080。  第一次打开网页会提示选择地图区域，配置语言，地图样式等。选择地图区域后，会要求输入一个密钥，已经注册的用户可以获取免费版的密钥。输入密钥后，进入下载进度页面。下载完后进入一个页面，然后点击一种样式进入地图页面。







