## 私有git仓库中使用go mod

假设公司git仓库地址是 gitlab.xxxx.com

1. 首先通go env查看当前的go环境变量，配置好相应的值，GOPROXY是go国内的代理服务器

```
set GO111MODULE=auto
set GOPRIVATE=gitlab.xxxx.com
set GOPROXY=https://mirrors.aliyun.com/goproxy/
```

2. 配置git的SSH免密登录

3. 修改 C:\Users\{用户}\.gitconfig, 将https协议转成git协议

```
[user]
	name = 你的git账号
	email = 你的git账号邮箱
[url "git@gitlab.xxxx.com:"]
    insteadOf = https://gitlab.xxxx.com/
 ```
 
 
