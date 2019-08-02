# 使用Filebeat收集k8s日志到elasticsearch

#### 基本原理：
将filebeat容器和应用容器放在同一个pod中，这样他们便可以共享一个volume。 在将应用的日志目录挂载在这个volume上，filebeat容器也有一个目录挂在这个volume上,这样filebeat便可以通过这个目录读取应用的日志文件。



下面的例子，在应用容器的日志在/work/logs目录下， 通过挂载myapp-logs这个volume将里面的文件共享到了filebeat容器的/logs目录。

#### filebeat容器的confimap文件

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: filebeat-config
  namespace: myapp
data:
  filebeat.yml: |
    filebeat.prospectors:
      - type: log
        paths:
          - /logs/error.log
    setup.template.name: "myapp"
    setup.template.pattern: "myapp-*"
    output.elasticsearch:
      index: myapp-%{+yyyy.MM.dd}
      hosts: ['elastisearch:9200']

````

#### k8s部署文件

````
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: filebeat
          image: docker.sz-shuwei.com/filebeat:6.4.2
          imagePullPolicy: IfNotPresent
          args: [
            "-c", "/home/filebeat-config/filebeat.yml",
          ]
          volumeMounts:
            - name: myapp-logs
              mountPath: /logs
            - name: filebeat-config
              mountPath: "/home/filebeat-config"
        - name: myapp
          image: myapp:latest
          imagePullPolicy: Always
          workingDir: /work
          ports:
            - containerPort: 35350
          command: ["myapp"]
          volumeMounts:
            - name: myapp-logs
              mountPath: /work/logs
      volumes:
        - name: myapp-logs
          emptyDir: {}
        - name: filebeat-config
          configMap:
            name: filebeat-config

````
