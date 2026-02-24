

## 【启动问题】

```
➜  /workspace git:(master) ✗ docker ps -a | grep -E "unhealthy|Exit"
9702edf5d28a   bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8       "/entrypoint.sh /run…"   4 minutes ago   Exited (255) 4 minutes ago                                                                               nm1
03d828020ca9   bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8       "/entrypoint.sh /run…"   4 minutes ago   Exited (255) 4 minutes ago                                                                               nm2
3f0301e76914   bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8   "/entrypoint.sh /run…"   4 minutes ago   Exited (255) 4 minutes ago                                                                               rm1
9764cd72e6b9   bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8   "/entrypoint.sh /run…"   4 minutes ago   Exited (255) 4 minutes ago                                                                               rm2
734c8328d233   bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh hdfs…"   4 minutes ago   Up 4 minutes (unhealthy)     9864/tcp                                                                    jn3
499823f568ce   bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh /run…"   4 minutes ago   Exited (1) 4 minutes ago                                                                                 nn2
7bdc9548dedb   bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh hdfs…"   4 minutes ago   Up 4 minutes (unhealthy)     9864/tcp                                                                    jn1
311f821a368d   bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh /run…"   4 minutes ago   Exited (1) 4 minutes ago                                                                                 nn1
3f28650cfcd3   bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh hdfs…"   4 minutes ago   Up 4 minutes (unhealthy)     9864/tcp                                                                    jn2
➜  /workspace git:(master) ✗ 
```


## 【启动报错】

```
# 查看 NameNode 错误
docker logs nn1 --tail 100

# 查看 ResourceManager 错误
docker logs rm1 --tail 100

# 查看 JournalNode 错误
docker logs jn1 --tail 100
```


```
➜  /workspace git:(master) ✗ docker logs rm1 --tail 100

Configuring core
 - Setting fs.defaultFS=hdfs://mycluster
Configuring hdfs
 - Setting dfs.namenode.rpc.address.mycluster.nn1=nn1:8020
 - Setting dfs.namenode.rpc.address.mycluster.nn2=nn2:8020
 - Setting dfs.ha.zookeeper.quorum=zk1:2181,zk2:2181,zk3:2181
 - Setting dfs.nameservices=mycluster
 - Setting dfs.ha.namenodes.mycluster=nn1,nn2
 - Setting dfs.namenode.shared.edits.dir=qjournal://jn1:8485;jn2:8485;jn3:8485/mycluster
Configuring yarn
 - Setting yarn.resourcemanager.ha.enabled=true
 - Setting yarn.resourcemanager.cluster.id=yarn-cluster
 - Setting yarn.resourcemanager.zk.address=zk1:2181,zk2:2181,zk3:2181
 - Setting yarn.resourcemanager.hostname.rm1=rm1
 - Setting yarn.resourcemanager.hostname.rm2=rm2
 - Setting yarn.resourcemanager.ha.rm.ids=rm1,rm2
Configuring httpfs
Configuring kms
Configuring mapred
Configuring for multihomed network
2026-02-24 20:26:25,739 INFO resourcemanager.ResourceManager: STARTUP_MSG: 

2026-02-24 20:26:26,740 INFO service.AbstractService: Service ResourceManager failed in state INITED
org.apache.hadoop.yarn.exceptions.YarnRuntimeException: Invalid configuration! Invalid value of yarn.resourcemanager.ha.rm-ids. Current value is null
HA mode requires atleast two RMs
        at org.apache.hadoop.yarn.conf.HAUtil.throwBadConfigurationException(HAUtil.java:45)
        at org.apache.hadoop.yarn.conf.HAUtil.verifyAndSetRMHAIdsList(HAUtil.java:120)
        at org.apache.hadoop.yarn.conf.HAUtil.verifyAndSetConfiguration(HAUtil.java:105)
        at org.apache.hadoop.yarn.server.resourcemanager.ResourceManager.serviceInit(ResourceManager.java:281)
        at org.apache.hadoop.service.AbstractService.init(AbstractService.java:164)
        at org.apache.hadoop.yarn.server.resourcemanager.ResourceManager.main(ResourceManager.java:1534)
2026-02-24 20:26:26,741 INFO resourcemanager.ResourceManager: Transitioning to standby state
2026-02-24 20:26:26,741 INFO resourcemanager.ResourceManager: Transitioned to standby state
2026-02-24 20:26:26,741 FATAL resourcemanager.ResourceManager: Error starting ResourceManager
org.apache.hadoop.yarn.exceptions.YarnRuntimeException: Invalid configuration! Invalid value of yarn.resourcemanager.ha.rm-ids. Current value is null
HA mode requires atleast two RMs
        at org.apache.hadoop.yarn.conf.HAUtil.throwBadConfigurationException(HAUtil.java:45)
        at org.apache.hadoop.yarn.conf.HAUtil.verifyAndSetRMHAIdsList(HAUtil.java:120)
        at org.apache.hadoop.yarn.conf.HAUtil.verifyAndSetConfiguration(HAUtil.java:105)
        at org.apache.hadoop.yarn.server.resourcemanager.ResourceManager.serviceInit(ResourceManager.java:281)
        at org.apache.hadoop.service.AbstractService.init(AbstractService.java:164)
        at org.apache.hadoop.yarn.server.resourcemanager.ResourceManager.main(ResourceManager.java:1534)
2026-02-24 20:26:26,743 INFO resourcemanager.ResourceManager: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down ResourceManager at 3f0301e76914/172.18.0.12
************************************************************/
```


```
➜  /workspace git:(master) ✗ docker logs nn1 --tail 100
java.io.IOException: Invalid configuration: a shared edits dir must not be specified if HA is not enabled.
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.<init>(FSNamesystem.java:796)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.<init>(FSNamesystem.java:737)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.format(NameNode.java:1184)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1649)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1759)
2026-02-24 20:26:23,448 ERROR namenode.NameNode: Failed to start namenode.
java.io.IOException: Invalid configuration: a shared edits dir must not be specified if HA is not enabled.
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.<init>(FSNamesystem.java:796)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.<init>(FSNamesystem.java:737)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.format(NameNode.java:1184)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1649)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1759)
2026-02-24 20:26:23,449 INFO util.ExitUtil: Exiting with status 1: java.io.IOException: Invalid configuration: a shared edits dir must not be specified if HA is not enabled.
2026-02-24 20:26:23,529 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at 311f821a368d/172.18.0.5
************************************************************/
2026-02-24 20:26:24,443 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = 311f821a368d/172.18.0.5
STARTUP_MSG:   args = []
STARTUP_MSG:   version = 3.2.1

...
...

2026-02-24 20:26:26,430 INFO namenode.FSNamesystem: Determined nameservice ID: mycluster
2026-02-24 20:26:26,430 INFO namenode.FSNamesystem: HA Enabled: false
2026-02-24 20:26:26,431 WARN namenode.FSNamesystem: Configured NNs:

2026-02-24 20:26:26,431 ERROR namenode.FSNamesystem: FSNamesystem initialization failed.
java.io.IOException: Invalid configuration: a shared edits dir must not be specified if HA is not enabled.
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.<init>(FSNamesystem.java:796)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFromDisk(FSNamesystem.java:712)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.loadNamesystem(NameNode.java:648)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.initialize(NameNode.java:710)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:953)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:926)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1692)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1759)
2026-02-24 20:26:26,436 INFO namenode.FSNamesystem: Stopping services started for active state
2026-02-24 20:26:26,436 INFO namenode.FSNamesystem: Stopping services started for standby state
2026-02-24 20:26:26,438 INFO handler.ContextHandler: Stopped o.e.j.w.WebAppContext@7486b455{/,null,UNAVAILABLE}{/hdfs}
2026-02-24 20:26:26,439 INFO server.AbstractConnector: Stopped ServerConnector@700fb871{HTTP/1.1,[http/1.1]}{0.0.0.0:9870}
2026-02-24 20:26:26,440 INFO handler.ContextHandler: Stopped o.e.j.s.ServletContextHandler@4b741d6d{/static,file:///opt/hadoop-3.2.1/share/hadoop/hdfs/webapps/static/,UNAVAILABLE}
2026-02-24 20:26:26,440 INFO handler.ContextHandler: Stopped o.e.j.s.ServletContextHandler@7d0b7e3c{/logs,file:///opt/hadoop-3.2.1/logs/,UNAVAILABLE}
2026-02-24 20:26:26,441 INFO impl.MetricsSystemImpl: Stopping NameNode metrics system...
2026-02-24 20:26:26,441 INFO impl.MetricsSystemImpl: NameNode metrics system stopped.
2026-02-24 20:26:26,441 INFO impl.MetricsSystemImpl: NameNode metrics system shutdown complete.
2026-02-24 20:26:26,441 ERROR namenode.NameNode: Failed to start namenode.
java.io.IOException: Invalid configuration: a shared edits dir must not be specified if HA is not enabled.
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.<init>(FSNamesystem.java:796)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFromDisk(FSNamesystem.java:712)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.loadNamesystem(NameNode.java:648)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.initialize(NameNode.java:710)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:953)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:926)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1692)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1759)
2026-02-24 20:26:26,442 INFO util.ExitUtil: Exiting with status 1: java.io.IOException: Invalid configuration: a shared edits dir must not be specified if HA is not enabled.
2026-02-24 20:26:26,443 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at 311f821a368d/172.18.0.5
************************************************************/
➜  /workspace git:(master) ✗ 
```

## 【failed】 放弃-nn1启动存在问题

