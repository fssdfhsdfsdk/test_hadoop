


```
➜  week1 git:(master) docker compose up -d
WARN[0000] /workspace/week1/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
[+] Running 18/18
 ✔ datanode Pulled                                                                                       16.7s 
 ✔ namenode Pulled                                                                                       16.7s 
[+] Running 3/3
 ✔ Network week1_hadoop-net   Created                                                                     0.1s 
 ✔ Container hadoop-namenode  Started                                                                     5.8s 
 ✔ Container hadoop-datanode  Started                                                                     1.4s 



➜  week1 git:(master) ✗ docker ps
CONTAINER ID   IMAGE                                                    COMMAND                  CREATED         STATUS                   PORTS                                       NAMES
398c30b4f733   bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8       "/entrypoint.sh /run…"   7 minutes ago   Up 7 minutes (healthy)   0.0.0.0:8042->8042/tcp, :::8042->8042/tcp   hadoop-nodemanager
0d304e9640cb   bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8   "/entrypoint.sh /run…"   7 minutes ago   Up 7 minutes (healthy)   0.0.0.0:8088->8088/tcp, :::8088->8088/tcp   hadoop-resourcemanager
28dd869b8f69   bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh /run…"   7 minutes ago   Up 7 minutes (healthy)   0.0.0.0:9864->9864/tcp, :::9864->9864/tcp   hadoop-datanode
396bdd22ed2b   bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh /run…"   7 minutes ago   Up 7 minutes (healthy)   0.0.0.0:9870->9870/tcp, :::9870->9870/tcp   hadoop-namenode

```


```
➜  week1 git:(master) ✗ docker compose logs -f namenode
WARN[0000] /workspace/week1/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
hadoop-namenode  | Configuring core
hadoop-namenode  |  - Setting fs.defaultFS=hdfs://namenode:9000
hadoop-namenode  | Configuring hdfs
hadoop-namenode  |  - Setting dfs.namenode.name.dir=file:///hadoop/nn
hadoop-namenode  | Configuring yarn
hadoop-namenode  |  - Setting yarn.resourcemanager.hostname=yarn-resourcemanager
hadoop-namenode  | Configuring httpfs
hadoop-namenode  | Configuring kms
hadoop-namenode  | Configuring mapred
hadoop-namenode  | Configuring for multihomed network
hadoop-namenode  | Formatting namenode name directory: /hadoop/nn
hadoop-namenode  | 2026-02-24 19:12:23,153 INFO namenode.NameNode: STARTUP_MSG: 
hadoop-namenode  | /************************************************************
hadoop-namenode  | STARTUP_MSG: Starting NameNode
hadoop-namenode  | STARTUP_MSG:   host = 220bf53c9e1e/172.18.0.2
hadoop-namenode  | STARTUP_MSG:   args = [-format, test-cluster]
hadoop-namenode  | STARTUP_MSG:   version = 3.2.1
hadoop-namenode  | STARTUP_MSG:   classpath =



➜  week1 git:(master) ✗ docker compose logs -f datanode
WARN[0000] /workspace/week1/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
hadoop-datanode  | Configuring core
hadoop-datanode  |  - Setting fs.defaultFS=hdfs://namenode:9000
hadoop-datanode  | Configuring hdfs
hadoop-datanode  |  - Setting dfs.datanode.data.dir=file:///hadoop/dn
hadoop-datanode  | Configuring yarn
hadoop-datanode  |  - Setting yarn.resourcemanager.hostname=yarn-resourcemanager
hadoop-datanode  |  - Setting yarn.nodemanager.hostname=yarn-nodemanager
hadoop-datanode  | Configuring httpfs
hadoop-datanode  | Configuring kms
hadoop-datanode  | Configuring mapred
hadoop-datanode  | Configuring for multihomed network
hadoop-datanode  | 2026-02-24 19:12:24,930 INFO datanode.DataNode: STARTUP_MSG: 
hadoop-datanode  | /************************************************************
hadoop-datanode  | STARTUP_MSG: Starting DataNode
hadoop-datanode  | STARTUP_MSG:   host = 89c6388b311e/172.18.0.3
hadoop-datanode  | STARTUP_MSG:   args = []
hadoop-datanode  | STARTUP_MSG:   version = 3.2.1
hadoop-datanode  | STARTUP_MSG:   classpath = /etc/hadoop:/opt/hadoop-3.2.1/

```

## 查看 HDFS 报告 (类似 df -h)

```
➜  week1 git:(master) ✗ docker exec -it hadoop-namenode bash
root@220bf53c9e1e:/# hdfs dfsadmin -report
Configured Capacity: 274877906944 (256 GB)
Present Capacity: 271212998656 (252.59 GB)
DFS Remaining: 271212994560 (252.59 GB)
DFS Used: 4096 (4 KB)
DFS Used%: 0.00%
Replicated Blocks:
        Under replicated blocks: 0
        Blocks with corrupt replicas: 0
        Missing blocks: 0
        Missing blocks (with replication factor 1): 0
        Low redundancy blocks with highest priority to recover: 0
        Pending deletion blocks: 0
Erasure Coded Block Groups: 
        Low redundancy block groups: 0
        Block groups with corrupt internal blocks: 0
        Missing block groups: 0
        Low redundancy blocks with highest priority to recover: 0
        Pending deletion blocks: 0

-------------------------------------------------
Live datanodes (1):

Name: 172.18.0.3:9866 (hadoop-datanode.week1_hadoop-net)
Hostname: 89c6388b311e
Decommission Status : Normal
Configured Capacity: 274877906944 (256 GB)
DFS Used: 4096 (4 KB)
Non DFS Used: 1605505024 (1.50 GB)
DFS Remaining: 271212994560 (252.59 GB)
DFS Used%: 0.00%
DFS Remaining%: 98.67%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Tue Feb 24 19:17:31 UTC 2026
Last Block Report: Tue Feb 24 19:12:37 UTC 2026
Num of Blocks: 0


root@220bf53c9e1e:/# 
```


## 创建目录、上传文件

```
root@220bf53c9e1e:/# hdfs dfs -mkdir -p /input
root@220bf53c9e1e:/# echo "Hello Hadoop Hello ZooKeeper" > /tmp/test.txt
root@220bf53c9e1e:/# hdfs dfs -put /tmp/test.txt /input/
2026-02-24 19:19:41,949 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
root@220bf53c9e1e:/# 
```

## 查看目录、查看文件

```
root@220bf53c9e1e:/# hdfs dfs -ls /input/
Found 1 items
-rw-r--r--   3 root supergroup         29 2026-02-24 19:19 /input/test.txt

root@220bf53c9e1e:/# hdfs dfs -cat /input/test.txt
2026-02-24 19:20:25,024 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
Hello Hadoop Hello ZooKeeper
root@220bf53c9e1e:/# 
```


## 【问题】工具位置

```
root@220bf53c9e1e:/# yarn jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar wordcount /input /output
JAR does not exist or is not a normal file: /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar
root@220bf53c9e1e:/# find /opt/hadoop -name "*mapreduce-examples*.jar" 2>/dev/null
root@220bf53c9e1e:/# find /opt/hadoop -name "*.jar" | head -20
find: '/opt/hadoop': No such file or directory
root@220bf53c9e1e:/# which hadoop
/opt/hadoop-3.2.1/bin//hadoop
root@220bf53c9e1e:/# find / -name "*mapreduce-examples*.jar" 2>/dev/null
/opt/hadoop-3.2.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar
/opt/hadoop-3.2.1/share/hadoop/mapreduce/sources/hadoop-mapreduce-examples-3.2.1-sources.jar
/opt/hadoop-3.2.1/share/hadoop/mapreduce/sources/hadoop-mapreduce-examples-3.2.1-test-sources.jar
root@220bf53c9e1e:/# 
```

## 【结果】运行计数结果

hdfs dfs -mkdir -p /input/xmls
hdfs dfs -put /etc/hadoop/*.xml /input/xmls
yarn jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar wordcount /input/xmls /output


```

2026-02-24 19:32:40,305 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2026-02-24 19:32:40,310 INFO mapred.MapTask: Starting flush of map output
2026-02-24 19:32:40,310 INFO mapred.LocalJobRunner: map task executor complete.
2026-02-24 19:32:40,324 WARN mapred.LocalJobRunner: job_local1732970437_0001
java.lang.Exception: java.io.FileNotFoundException: Path is not a file: /input/xmls
        at org.apache.hadoop.hdfs.server.namenode.INodeFile.valueOf(INodeFile.java:90)
        at org.apache.hadoop.hdfs.server.namenode.INodeFile.valueOf(INodeFile.java:76)

2026-02-24 19:32:41,189 INFO mapreduce.Job: Job job_local1732970437_0001 running in uber mode : false
2026-02-24 19:32:41,189 INFO mapreduce.Job:  map 100% reduce 0%
2026-02-24 19:32:41,191 INFO mapreduce.Job: Job job_local1732970437_0001 failed with state FAILED due to: NA
2026-02-24 19:32:41,194 INFO mapreduce.Job: Counters: 24
        File System Counters
                FILE: Number of bytes read=316790
                FILE: Number of bytes written=842101
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=29
                HDFS: Number of bytes written=0
                HDFS: Number of read operations=5
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=1
                HDFS: Number of bytes read erasure-coded=0
        Map-Reduce Framework
                Map input records=1
                Map output records=4
                Map output bytes=45
                Map output materialized bytes=47
                Input split bytes=100
                Combine input records=4
                Combine output records=3
                Spilled Records=3
                Failed Shuffles=0
                Merged Map outputs=0
                GC time elapsed (ms)=0
                Total committed heap usage (bytes)=1552941056
        File Input Format Counters 
                Bytes Read=29
root@220bf53c9e1e:/# 

```


```
root@220bf53c9e1e:/# hdfs dfs -rm -r /output
Deleted /output
root@220bf53c9e1e:/# hdfs dfs -ls /input/xmls/
Found 9 items
-rw-r--r--   3 root supergroup       8260 2026-02-24 19:27 /input/xmls/capacity-scheduler.xml
-rw-r--r--   3 root supergroup        856 2026-02-24 19:27 /input/xmls/core-site.xml
-rw-r--r--   3 root supergroup      11392 2026-02-24 19:27 /input/xmls/hadoop-policy.xml
-rw-r--r--   3 root supergroup       1379 2026-02-24 19:27 /input/xmls/hdfs-site.xml
-rw-r--r--   3 root supergroup        620 2026-02-24 19:27 /input/xmls/httpfs-site.xml
-rw-r--r--   3 root supergroup       3518 2026-02-24 19:27 /input/xmls/kms-acls.xml
-rw-r--r--   3 root supergroup        682 2026-02-24 19:27 /input/xmls/kms-site.xml
-rw-r--r--   3 root supergroup        841 2026-02-24 19:27 /input/xmls/mapred-site.xml
-rw-r--r--   3 root supergroup       1130 2026-02-24 19:27 /input/xmls/yarn-site.xml
root@220bf53c9e1e:/# yarn jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar wordcount /input/xmls /output

...
...

root@220bf53c9e1e:/# hdfs dfs -ls /output/
Found 2 items
-rw-r--r--   3 root supergroup          0 2026-02-24 19:38 /output/_SUCCESS
-rw-r--r--   3 root supergroup      10418 2026-02-24 19:38 /output/part-r-00000
```


```
root@220bf53c9e1e:/# hdfs dfs -cat /output/part-r-00000
2026-02-24 19:40:52,016 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
"*"     21
"AS     9
"License");     9
"alice,bob      21
"clumping"      1
(ASF)   1
(root   1
(the    9
-->     18
-1      1
-1,     1
0.0     1
1-MAX_INT.      1
1.      1
1.0.    1
2.0     9
40      2
40+20=60        1
:       2
<!--    18
</configuration>        9
</description>  33
</name> 2
</property>     57
<?xml   8
<?xml-stylesheet        4
<configuration> 9
<description>   31
<description>ACL        25
<description>Default    1
<name>default.key.acl.DECRYPT_EEK</name>        1


will    12
with    28
work    1
writing,        9
you     10
zero    2
root@220bf53c9e1e:/# 
```


## 【问题】服务异常

```
9870页面正常；打开 8088，页面只有 “read ECONNRESET 或 socket hang up”；

➜  week1 git:(master) ✗ lsof -i :8088
COMMAND    PID USER   FD   TYPE     DEVICE SIZE/OFF NODE NAME
docker-pr 4700 root    4u  IPv4 2321090659      0t0  TCP *:omniorb (LISTEN)
docker-pr 4707 root    4u  IPv6 2321084652      0t0  TCP *:omniorb (LIST

➜  week1 git:(master) ✗ docker compose logs namenode | grep ResourceManager
WARN[0000] /workspace/week1/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
➜  week1 git:(master) ✗ EN)
```
从你的输出来看：

1.  **8088 端口已被 Docker 代理占用** → 端口映射没问题
2.  **9870 (HDFS) 正常** → NameNode 服务正常
3.  **8088 (YARN) 无法访问** → **ResourceManager 未启动**

**根本原因：** `bde2020/hadoop-namenode` 镜像 **默认只启动 HDFS NameNode**，不启动 YARN ResourceManager！需要额外配置或更换镜像方案。


在 NameNode 容器中手动启动 YARN（推荐，快速验证）

```
➜  week1 git:(master) ✗ docker exec -it hadoop-namenode bash
root@220bf53c9e1e:/# jps
4849 Jps
258 NameNode


root@220bf53c9e1e:/# $HADOOP_HOME/bin/yarn --daemon start resourcemanager
root@220bf53c9e1e:/# $HADOOP_HOME/bin/yarn --daemon start nodemanager
root@220bf53c9e1e:/# jps
5312 NodeManager
258 NameNode
5459 Jps
4975 ResourceManager
```

## 【网页】

**👀 观察点：**

-   打开浏览器访问 `http://localhost:9870` (NameNode UI)。
-   查看 **Live Nodes** 是否为 1。
-   查看 **Capacity** 和 **Used** 空间变化。
-   **思考：** 此时数据副本数是多少？(默认 3 副本，但只有 1 个 DN，Hadoop 会自动调整为 1 副本，观察 `Under Replicated Blocks`)。

打开浏览器访问 `http://localhost:8088` (ResourceManager UI)。

**👀 观察点：**

-   查看 **Nodes** 页面，确认 NodeManager 是否活跃。
-   此时还没有任务，所以 **Applications** 为空。


## 【清理】清理环境


```
➜  week1 git:(master) ✗ docker compose down -v  # -v 会删除卷，重置数据
WARN[0000] /workspace/week1/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
[+] Running 1/2
 ✔ Container hadoop-datanode  Removed                                                                     [+] Running 3/3            10.3s 
 ✔ Container hadoop-datanode  Removed                                                               10.3s 
 ✔ Container hadoop-namenode  Removed                                                               10.8s 
 ✔ Network week1_hadoop-net   Removed 
 ```