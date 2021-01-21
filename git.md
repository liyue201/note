## git

### http代理

设置（若v2ray socks端口是1080，则http端口加1是1081）
```
git config --global http.proxy http://127.0.0.1:1081
```

取消
```
git config --global --unset http.proxy
```


###记住用户名和密码

进入项目目录
```
git config --global credential.helper store
git pull 
```

清除用户名和密码
```
git config --global credential.helper wincred
git credential-manager uninstall
```

