
### 一、前提准备

##### 1.1 下载 jdk-8u211-linux-x64.tar.gz  
拷贝到 hd-container 目录下(利用Dockerfile制作镜像的时候要用到,注意两边的版本号一致)
##### 1.2 下载 apache-hive-3.1.2-bin.tar.gz  
   `wget http://apache.01link.hk/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz`   
   拷贝到 目录 `build-env/etc` 下，并解压 
##### 1.3 下载 hadoop-3.1.2.tar.gz   
`wget https://archive.apache.org/dist/hadoop/common/hadoop-3.1.2/hadoop-3.1.2.tar.gz`  
拷贝到 目录 `build-env/etc` 下，并解压 
##### 1.4 制作 docker 构建
  ```bash
  git clone https://github.com/qinrui777/bigdata-docker.git
  cd bigdata-docker
  cd  hd-container &&  docker build -t hd-container:1.X .
  ```
  > 等待5min

### 二、节点预览

| node role |    hostname  |  container IP |   components role  |
| :---------| ------------|  :------     |   -------| 
|    master | hadoop-master1 | 172.19.0.10 | NameNode,NameNode, hive server
|   slave1  | hadoop-slave1  | 172.19.0.11 | DataNode
|    slave2  | hadoop-slave2 |  172.19.0.12| NodeManager

### 三、启动Hadoop运行环境

##### 3.1 启动容器
```bash
#mac os
cd build-env && docker-compose -p bd -f mac-docker-compose-hive.yml up -d

# windowns
cd build-env && docker-compose -p bd -f win-docker-compose-hive.yml up -d
```

##### 3.2 进入容器，初始化环境
在 **master node** 上执行

```bash
# 先进入 master 容器
docker exec -it bd_federation-master1_1 bash
# 定义环境变量
source /opt/script/bigdata_env.sh
# 格式化，并启动 master
hdfs namenode -format
sh /opt/script/start-components.sh master
```

分别在两个 **slave node** 上执行

```sh
# 初始化环境 
source /opt/script/bigdata_env.sh
# 启动slave
sh /opt/script/start-components.sh slave
```

##### 3.3 初始化hive

在 **master node** 上执行
```bash
hadoop fs -mkdir -p /user/hive/warehouse
hadoop fs -chown hive:hive /user/hive
hadoop fs -chown hive:hive /user/hive/warehouse
schematool -dbType derby -initSchema

# hive 启动命令 ( 查找日志路径 find / -name hive.log )
nohup hive --service metastore &
nohup hiveserver2 &
```

hive server2 连接测试, 在 **master node** 上执行   
`beeline -u jdbc:hive2://localhost:10000/default -n hive`

交互命令行中，测试 hive

```
create table test(id string, name string, age int)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```

`insert into table test values('1','q1',33);`

`LOAD DATA INPATH '/tmp/test.txt' INTO TABLE test;`

2/ 分区关键字 PARTITIONED BY
```sql
create table t_4(ip string,url string,staylong int)
partitioned by (day string)
row format delimited
fields terminated by ',';
```

导入数据到不同的分区目录：
```bash
load data local inpath '/root/weblog.1' into table t_4 partition(day='2017-04-08');
load data local inpath '/root/weblog.2' into table t_4 partition(day='2017-04-09');
```

### 清理环境

```bash
cd build-env # 进入 mac-docker-compose-hive.yml 所在的目录
docker-compose -p bd -f mac-docker-compose-hive.yml down -v
```

---
### 参考1:  关于docker
善意提醒提高docker container 的内存大小，被坑了好久。

### 参考2: hadoop 常用命令

`hdfs namenode -format`

`hdfs --daemon start namenode`

`hdfs --daemon start datanode`

`yarn --daemon start resourcemanager`

`yarn --daemon start nodemanager`

`hadoop fs -mkdir -p /user/hive/warehouse`

### 参考3: hive on hadoop 3 feature 
https://mathsigit.github.io/blog_page/2017/11/16/hole-of-submitting-mr-of-hadoop300RC0/ 
