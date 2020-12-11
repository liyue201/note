## IPFS私网部署
### 节点准备
准备3台主机
- A: 192.168.3.101
- B: 192.168.3.102
- C: 192.168.3.103

### 安装IPFS
参考： https://docs.ipfs.io/recent-releases/go-ipfs-0-7/install/#linux  
在3台主机上分别安装IPFS
```
wget https://dist.ipfs.io/go-ipfs/v0.7.0/go-ipfs_v0.7.0_linux-amd64.tar.gz
tar -xvzf go-ipfs_v0.7.0_linux-amd64.tar.gz
cd go-ipfs
./install.sh
ipfs --version
```
初始化仓库目录，会自动生成目录` ~/.ipfs`
```
ipfs init
```

### 生成密钥
参考： https://my.oschina.net/u/4410065/blog/3723123

在A节点上，下载工具，生成密钥文件swarm.key

```
go get -u github.com/Kubuxu/go-ipfs-swarm-key-gen/ipfs-swarm-key-gen
cd github.com/Kubuxu/go-ipfs-swarm-key-gen/ipfs-swarm-key-gen
go build 
./ipfs-swarm-key-gen  > swarm.key
```

将密钥文件swarm.key分别拷贝到A、B、C节点的配置目录`~/.ipfs/`下
```
 scp swarm.key root@192.168.3.101:/root/.ipfs
 scp swarm.key root@192.168.3.102:/root/.ipfs
 scp swarm.key root@192.168.3.103:/root/.ipfs
```

### 启动节点
在 A、B、C上删除默认的连接节点。添加A节点作为启动连接节点。
```
ipfs bootstrap rm --all　
ipfs bootstrap add /ip4/192.168.3.101/ipfs/QmUpwjfX6gedCEeh6ncRfshXnDuwSbvZh1uXpriMy1kFnH
```

启动 A、B、C 节点

```
nohup ipfs daemon &> ipfs.log &
```

查看对等网络,在A中执行
```
root@test-201:~# ipfs swarm  peers
/ip4/192.168.3.102/tcp/4001/p2p/12D3KooWRRwB3zqGvGT6cXckhkBe81CTf4duNxaFRPknkUyK5Bxa
/ip4/192.168.3.103/tcp/4001/p2p/12D3KooWSsAV5K1my2NzhBB2QGixEMj8HuRs9rZa8YTvFb4WxLRi
```

### 部署webui， 

在A节点安装webui，指定端口3001
```
https://github.com/ipfs-shipyard/ipfs-webui
cd ipfs-webui
npm install -g cnpm --registry=https://registry.npm.taobao.org	
cnpm install
PORT=3001 npm start
```
或者使用docker
```
docker run -d --name webui -p 3001:3000  garychen/ipfs-webui:latest
```

打开web
http://192.168.3.101:3001

报错，根据提示修改配置 `~/.ipfs/config`,以下3处
```
{
  "API": {
    "HTTPHeaders": {
      "Access-Control-Allow-Methods": [  //1.这里
        "PUT",
        "POST"
      ],
      "Access-Control-Allow-Origin": [  
        "http://192.168.3.101:3001"   // 2.这里
      ]
    }
  },
  "Addresses": {
    "API": "/ip4/0.0.0.0/tcp/5001",    // 3.这里
    "Announce": [],
    "Gateway": "/ip4/127.0.0.1/tcp/8080",
    "NoAnnounce": [],
    "Swarm": [
      "/ip4/0.0.0.0/tcp/4001",
      "/ip6/::/tcp/4001",
      "/ip4/0.0.0.0/udp/4001/quic",
      "/ip6/::/udp/4001/quic"
    ]
  },
   
  // 省略
   
}

```

重启ipfs
```
nohup ipfs daemon &> ipfs.log &
```
在web页面配置节点地址 `/ip4/192.168.3.101/tcp/5001`


