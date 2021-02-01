
# lotus多节点私网部署

### 编译代码

基于官方主网，修改了一些参数，支持2k、256M、32G、64G扇区。

```
git clone https://github.com/blockchain-lib/lotus
cd lotus 
git checkout net/private-1.4.1
make 
sudo make install
make lotus-seed
sudo cp lotus-seed /usr/local/bin
```

### 创世节点

环境变量
```
export LOTUS_SKIP_GENESIS_CHECK=_yes_
export LOTUS_DIR=/lotus/
export LOTUS_PATH=/lotus/lotus
export LOTUS_MINER_PATH=/lotus/lotusminer
```

创世节点
```
lotus fetch-params 2048

lotus-seed --sector-dir=$LOTUS_DIR/.genesis-sectors  pre-seal --sector-size 2KiB

lotus-seed genesis new localnet.json

lotus-seed genesis add-miner localnet.json $LOTUS_DIR/.genesis-sectors/pre-seal-f01000.json

nohup lotus daemon --lotus-make-genesis=private.car --genesis-template=localnet.json --bootstrap=false &> lotus.log &
```

修改p2p端口为固定端口，比如55555, `lotus/config.toml `，然后，重启lotus daemon
```
[Libp2p]
  ListenAddresses = ["/ip4/0.0.0.0/tcp/55555", "/ip6/::/tcp/0"]
```

部署创世矿工
```
lotus wallet import --as-default $LOTUS_DIR/.genesis-sectors/pre-seal-f01000.key

lotus-miner init --genesis-miner --actor=f01000 --sector-size=2KiB --pre-sealed-sectors=$LOTUS_DIR/.genesis-sectors --pre-sealed-metadata=$LOTUS_DIR/.genesis-sectors/pre-seal-f01000.json --nosync

nohup lotus-miner run --nosync &> miner.log &
```

查看节点p2p地址，其他节点连接时候使用
```
$ lotus net listen

/ip4/192.168.6.55/tcp/55555/p2p/12D3KooWSr6MuGscVkqvrdXjsVDwB7whQbFj94SmzJs6MStdhYsB
/ip4/127.0.0.1/tcp/55555/p2p/12D3KooWSr6MuGscVkqvrdXjsVDwB7whQbFj94SmzJs6MStdhYsB
/ip6/::1/tcp/42457/p2p/12D3KooWSr6MuGscVkqvrdXjsVDwB7whQbFj94SmzJs6MStdhYsB
```

查看钱包
```
$ lotus wallet list
Address                                                                                 Balance                          Nonce  Default  
f3q3ziuyfbwsw5yxa6tnv4vxejkvqlfbdwib3klyvrbhzamqx2gikafnp6irpfjt25mcyvn36flycfhpdvv37q  49999999.999409530200323975 FIL  5      X 
```

### 其他节点

拷贝上面生成的private.car文件到本机，启动节点
```
lotus daemon --genesis=private.car  --bootstrap=false
```

连接创世节点，使用上面` lotus net listen`，命令返回的地址。如果有公网ip，换成公网ip。
```
lotus net connect /ip4/192.168.6.55/tcp/55555/p2p/12D3KooWSr6MuGscVkqvrdXjsVDwB7whQbFj94SmzJs6MStdhYsB
```
查看连接情况
```
$ lotus net peers

12D3KooWSr6MuGscVkqvrdXjsVDwB7whQbFj94SmzJs6MStdhYsB, [/ip4/192.168.6.55/tcp/55555]

```

新建钱包地址
```
$ lotus wallet new bls

f3slo5iisbjgnndg5gygkpqpfv3xotjjqir4scs7bep5gaxfpgahj4y2bjlk7mjsrm6iomyr3rwfm4onzfkbia
```
查看地址余额

```
$ lotus wallet list
Address                                                                                 Balance  Nonce  Default  
f3slo5iisbjgnndg5gygkpqpfv3xotjjqir4scs7bep5gaxfpgahj4y2bjlk7mjsrm6iomyr3rwfm4onzfkbia  0 FIL    0      X  
```

通过创世节点给新建的地址转账
```
$ lotus send f3slo5iisbjgnndg5gygkpqpfv3xotjjqir4scs7bep5gaxfpgahj4y2bjlk7mjsrm6iomyr3rwfm4onzfkbia 1000
bafy2bzaceanzlldrirt2yo7huslkidzptgyosdww7o3wpubxp3e3puysrf5gm
```

查看余额变化
```
$ lotus wallet list
Address                                                                                 Balance  Nonce  Default  
f3slo5iisbjgnndg5gygkpqpfv3xotjjqir4scs7bep5gaxfpgahj4y2bjlk7mjsrm6iomyr3rwfm4onzfkbia  1000 FIL    0      X  
```
### 参考 
https://docs.filecoin.io/build/local-devnet/#devnet-with-vanilla-lotus-binaries

