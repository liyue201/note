

## Spring Boot集成SkyWalking Agent


下载skywalking 二进制安装包，解压获取agent目录的文件 
```
https://mirror.bit.edu.cn/apache/skywalking/6.6.0/apache-skywalking-apm-6.6.0.tar.gz
```

### linux环境

SkyWalking Agent配置

```
export SW_AGENT_NAME=demo-application # 配置 Agent 名字。一般来说，我们直接使用 Spring Boot 项目的 `spring.application.name` 。
export SW_AGENT_COLLECTOR_BACKEND_SERVICES=127.0.0.1:11800 # 配置 Collector 地址。
export SW_AGENT_SPAN_LIMIT=2000 # 配置链路的最大 Span 数量。一般情况下，不需要配置，默认为 300 。主要考虑，有些新上 SkyWalking Agent 的项目，代码可能比较糟糕。
export JAVA_AGENT=-javaagent:/Users/yunai/skywalking/apache-skywalking-apm-bin/agent/skywalking-agent.jar # SkyWalking Agent jar 地址。
```

Jar 启动
```
java -jar $JAVA_AGENT -jar lab-39-demo-2.2.2.RELEASE.jar

```

### windows环境

SkyWalking Agent配置
```
set SW_AGENT_NAME=demo-application
set SW_AGENT_COLLECTOR_BACKEND_SERVICES=10.0.101.67:11800 
set SW_AGENT_SPAN_LIMIT=2000 
set JAVA_AGENT=-javaagent:F:\\code\\java\\SpringBoot-Labs\\lab-39\\lab-39-demo\\agent\\skywalking-agent.jar
```

Jar 启动
```
java %JAVA_AGENT% -Dskywalking.agent.service_name=%SW_AGENT_NAME% -Dskywalking.collector.backend_service=%SW_AGENT_COLLECTOR_BACKEND_SERVICES% -Dskywalking.agent.span_limit_per_segment=%SW_AGENT_SPAN_LIMIT% -jar lab-39-demo-2.2.2.RELEASE.jar

```



参考 ： http://skywalking.apache.org/zh/blog/2020-04-19-skywalking-quick-start.html
