## 本地单实例搭建OLAP方案所需组件
### 初始化docker集群
```shell
docker swarm init
```
#### [Docker官网对于swarm init的描述](https://docs.docker.com/engine/reference/commandline/swarm_init/)

### 创建一个局域网
```shell
docker network create --attachable --driver overlay --subnet 10.1.0.0/16 --ip-range 10.1.0.0/24 tools
```
#### [Docker官网对于 network create的描述](https://docs.docker.com/engine/reference/commandline/network_create/)

### 创建Mysql 5.7容器实例
```shell
docker run -d --name mysql --network tools -p 3306:3306 --restart unless-stopped -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root@local mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci
```
#### [Docker 对于run的描述](https://docs.docker.com/engine/reference/commandline/run/)

### 创建Pulsar 2.7.1的容器实例
```shell
docker run -d --name pulsar-2.7 --network tools -m 3g -p 6650:6650 -p 6680:8080 --restart unless-stopped -v  pulsar-data:/pulsar/data -v pulsar-conf:/pulsar/conf apachepulsar/pulsar:2.7.1 ./bin/pulsar standalone
```
### 创建Flink 1.1.2（目前最新） 的容器实例
```shell
FLINK_PROPERTIES="jobmanager.rpc.address: jobmanager"
```
#### [Flink官网对于运行在docker的描述](https://ci.apache.org/projects/flink/flink-docs-master/docs/deployment/resource-providers/standalone/docker/)
#### 创建JobManager
```shell
docker run -d --name=jobmanager --network tools --publish 8081:8081 --env FLINK_PROPERTIES="${FLINK_PROPERTIES}" flink:latest jobmanager
```

#### 创建一个或多个TaskManager
```shell
docker run -d --name=taskmanager --network tools --env FLINK_PROPERTIES="${FLINK_PROPERTIES}" flink:latest taskmanager
```

### 使用Docker开发镜像编译Doris
#### 使用现成镜像
```shell
docker pull apachedoris/doris-dev:build-env
```

#### 运行镜像
```shell
docker run -it apachedoris/doris-dev:build-env
```

####



