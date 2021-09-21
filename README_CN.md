#### `Promethues 部署指南`

中文 | [English](README.md)

#### `什么是Prometheus?`
    Prometheus是一个开源的系统监控和警报工具包，最初在SoundCloud建立。

### `功能`
Prometheus的主要功能:

* 一个多维的数据模型，其中的时间序列数据由指标名称和键/值对识别
* PromQL是一种灵活的查询语言，可以利用这种维度。
* 不依赖分布式存储；单个服务器节点是自主的
* 通过HTTP的拉动模式进行时间序列的收集
* 通过中间网关支持时间序列的推送
* 通过服务发现或静态配置来发现目标
* 支持多种模式的图表和仪表盘制作


### `在docker中部署Prometheus`
``` docker
docker run -d \
    -p 9090:9090 \
    --name=prometheus \
    --name prometheus \
    -v ~/tmp/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```

### `在docker中部署Jenkins`

* `Jenkins: 一个开源的自动化服务器，使世界各地的开发者能够可靠地构建、测试和部署他们的软件。`
  
##### run command
``` docker
docker run -d -p 8080:8080 -p 5000:5000 \
  --name jenkins \
  -v $(which docker):/usr/bin/docker\
  -v jenkins_home:/var/jenkins_home \
  --privileged=true \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:latest
```
###### 必须把jenkins用户添加到docker用户组
```shell
sudo usermod -aG docker jenkins
sudo chmod 777 /var/run/docker.sock
```
###### 启动Jenkins服务和刷新用户组.
```shell
docker restart jenkins
```
##### 或者重启服务器
```shell
sudo reboot
```

### `在docker中部署Granafa`
* `Grafana允许你查询、可视化、提醒和了解你的指标，无论它们存储在哪里。创建、探索并与你的团队分享仪表盘，培养数据驱动文化。`

```docker
docker run -d -p 3001:3000 \
  --name granfana \
  grafana/grafana
```


### `在docker中部署Cadvisor`
* `cAdvisor (Container Advisor) 
为容器用户提供了对其运行中的容器的资源使用和性能特征的了解。它是一个运行中的守护程序，收集、汇总、处理和输出运行中的容器的信息。具体来说，它为每个容器保存资源隔离参数、历史资源使用情况、完整的历史资源使用直方图和网络统计数据。这些数据是按容器和机器范围导出的。`
  
```docker
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8082:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  google/cadvisor:latest
```

### `Prometheus导出器`

```docker
docker run -d -p 9100:9100 \
  -v "/:/hostfs" \
  --name=node-exporter \
  prom/node-exporter \
  --path.rootfs=/hostfs
```

<font color=Green>Node Export</font>
```text
node_exporter被设计用来监控主机系统。不建议将其作为Docker容器部署。
```

[Node Exporter docs](https://github.com/prometheus/node_exporter)

* <font color=Green>Jenkins Prometheus metrics </font>
  
    `Expose Jenkins metrics in prometheus format`
    [Jenkins Prometheus metrics docs](https://plugins.jenkins.io/prometheus/)

### `创建PostgreSQL容器`
```
docker run -d --name postgres -e POSTGRES_PASSWORD=12345678 -e POSTGRES_DBNAME=emperor_production -e POSTGRES_USERNAME=emperor -p 5432:5432 postgres:11
```
* 显示容器的IP地址
```
docker inspect -f "{{ .NetworkSettings.IPAddress }}" $container_name
```
