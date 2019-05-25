## k8s通过ingress暴露dashborad

### 部署nginx-ingress

参考：
```
https://www.cnblogs.com/panwenbin-logs/p/9915927.html
https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.20.0/deploy/mandatory.yaml
```

下载镜像
```
$ docker pull registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/defaultbackend-amd64:1.5 
$ docker pull registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/nginx-ingress-controller:0.20.0
```
 
下载yaml文件并更新mandatory.yaml中的镜像地址
```
$ mkdir ingress-nginx
$ cd ingress-nginx
$ wget  https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.20.0/deploy/mandatory.yaml
$ wget  https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml         
```
替换defaultbackend-amd64镜像地址
```
$ sed -i 's#k8s.gcr.io/defaultbackend-amd64#registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/defaultbackend-amd64#g' mandatory.yaml  
$  sed -i 's#quay.io/kubernetes-ingress-controller/nginx-ingress-controller#registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/nginx-ingress-controller#g' mandatory.yaml 
```

检查替换结果
```
$ grep image mandatory.yaml  
     # Any image is permissible as long as:
     image: registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/defaultbackend-amd64:1.5
     image: registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/nginx-ingress-controller:0.20.0
```		  
		  
修改service-nodeport.yaml文件，添加NodePort端口，默认为随机端口
```	
$ vim service-nodeport.yaml

apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
      nodePort: 32080  #http
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
      nodePort: 32443  #https
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
```	

部署nginx-ingress-controller
```	
$ kubectl apply -f mandatory.yaml 
$ kubectl apply -f service-nodeport.yaml 
```	

### 部署 dashboard

参考：
```	
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/ 
https://my.oschina.net/u/2306127/blog/1930169?from=timeline&isappinstalled=0
```	

下载镜像
```	
docker pull siriuszg/kubernetes-dashboard-amd64:v1.10.1
```	

下载部署文件
```	
wget https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
```	
修改kubernetes-dashboard.yaml 镜像为
```	
siriuszg/kubernetes-dashboard-amd64:v1.10.1
```	
部署
```	
$ kubectl apply -f  kubernetes-dashboard.yaml
```	

配置ingress-dashboard 证书
```	
$ openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout ./tls.key -out ./tls.crt -subj "/CN=10.102.20.127"
$ kubectl -n kube-system create secret tls k8s-dashboard-ingress-secret --key ./tls.key --cert ./tls.crt
```	

配置ingress-dashboard
```	
$ vim ingress-dashboard.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-dashboard
  namespace: kube-system
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/secure-backends: "true"
    kubernets.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - k8s-dashboard.net
    secretName: k8s-dashboard-ingress-secret
  rules:
  - host: k8s-dashboard.net
    http:
      paths:
      - path:
        backend:
          serviceName: kubernetes-dashboard
          servicePort: 443
```	  

部署ingress-dashboard
```	
$ kubectl apply -n kube-system -f ingress-dashboard.yaml 
```	

建立授权账号及Token
建立Dashboard访问的授权账号，将下面的内容保存为文件，如dashboard-rbac.yaml。
```	
$ vim dashboard-rbac.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard
  namespace: kube-system

---

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dashboard
subjects:
  - kind: ServiceAccount
    name: dashboard
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
  
```	
  
部署授权账号
```	
$ kubectl create -f dashboard-rbac.yaml
```	

查看token，复制token到dashboard上登录 
```	
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep dashboard-token | awk '{print $1}')
```	

```	
Name:         dashboard-token-8hdmh
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard
              kubernetes.io/service-account.uid: 21d7f8d3-7ea0-11e9-ba0f-005056b2b335

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtdG9rZW4tOGhkbWgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMjFkN2Y4ZDMtN2VhMC0xMWU5LWJhMGYtMDA1MDU2YjJiMzM1Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZCJ9.uRcwL77lB4ZrKTarZE3YivfNt0U5t4swp3GffVaeFu-tdc53PDmG-zSORfw7PQV1nfLVbzYtMTM56nzwqpMu-SNcD_BZTgE-y94kAdtRCVvPiUw_HgfP_HmQKtN81o1ih4InClyDbaMDxwz91v_G5bDjE3bPS1Fqt323cBgmuNXcuPDDXwUiq2RRIEERakmSssSOoPXZopnjDotIAYtj-elH-QOd05pNjekVGMW29Lruz2l_67Lmk-7ITCqc8yCdBaTdbxk99pnSYp0hk5triEmalMY1WgOfRVs1OfzPhPScFmhhBn1ogxi1JhDt2-rGfBWKbP2I1cArru4ZRfVZHg

```	
