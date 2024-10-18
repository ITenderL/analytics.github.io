# HDFS

``` shell
# 一键启动hdfs集群
start-dfs.sh
# 一键关闭hdfs集群
stop-dfs.sh

# 如果遇到命令未找到的错误，表明环境变量未配置好，可以以绝对路径执行
/export/server/hadoop/sbin/start-dfs.sh
/export/server/hadoop/sbin/stop-dfs.sh

# 将配置好的core-site.xml和hdfs-site.xml分发到node2和node3
# 重启Hadoop HDFS集群（先stop-dfs.sh，后start-dfs.sh）
# 停止系统的NFS相关进程
a. systemctl stop nfs; systemctl disable nfs    # 关闭系统nfs并关闭其开机自启
b. yum remove -y rpcbind    #卸载系统自带rpcbind
# 启动portmap（HDFS自带的rpcbind功能）（必须以root执行）：
hdfs --daemon start portmap
# 启动nfs（HDFS自带的nfs功能）（必须以hadoop用户执行）：
hdfs --daemon start nfs3


# 启动yarn
$HADOOP_HOME/sbin/start-yarn.sh
# 历史服务器
$HADOOP_HOME/bin/mapred --daemon start historyserver

#  并通过命令启动它即可 （部署环节会使用到）
$HADOOP_YARN_HOME/sbin/yarn-daemon.sh start proxyserver

# 常用的进程启动命令如下：
# 一键启动YARN集群： 
$HADOOP_HOME/sbin/start-yarn.sh
# 会基于yarn-site.xml中配置的yarn.resourcemanager.hostname来决定在哪台机器上启动resourcemanager
# 会基于workers文件配置的主机启动NodeManager
# 一键停止YARN集群： 
$HADOOP_HOME/sbin/stop-yarn.sh
# 在当前机器，单独启动或停止进程
$HADOOP_HOME/bin/yarn --daemon start|stop resourcemanager|nodemanager|proxyserver
# start和stop决定启动和停止
# 可控制resourcemanager、nodemanager、proxyserver三种进程
# 历史服务器启动和停止
$HADOOP_HOME/bin/mapred --daemon start|stop historyserver

# 分发
scp -r hadoop-3.3.4 node2:`pwd`/
scp -r hadoop-3.3.4 node3:`pwd`/

```

# Hive

``` shell
# 保Hive文件夹所属为hadoop用户
# 创建一个hive的日志文件夹： 
mkdir /export/server/hive/logs
# 启动元数据管理服务（必须启动，否则无法工作）
#前台启动：
bin/hive --service metastore 
#后台启动：
nohup bin/hive --service metastore >> logs/metastore.log 2>&1 &
# 启动客户端，二选一（当前先选择Hive Shell方式）
# Hive Shell方式（可以直接写SQL）： 
bin/hive
#Hive ThriftServer方式（不可直接写SQL，需要外部客户端链接使用）： 
bin/hive --service hiveserver2 >> logs/hiveserver2.log 2>&1 &


nohup hive --service hiveserver2 &
nohup hive --service metastore &

-- 在Hive的MySQL元数据库中执行，hive乱码
use hive;
-- 1.修改字段注释字符集
alter table COLUMNS_V2 modify column COMMENT varchar(256) character set utf8;
-- 2.修改表注释字符集
alter table TABLE_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;
-- 3.修改分区表参数，以支持分区键能够用中文表示
alter table PARTITION_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;
alter table PARTITION_KEYS modify column PKEY_COMMENT varchar(4000) character set utf8;
-- 4.修改索引注解
alter table INDEX_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;

```

# Spark

``` shell
# 历史服务器
sbin/start-history-server.sh
# 启动全部master和worker
sbin/start-all.sh

# 或者可以一个个启动:
# 启动当前机器的master
sbin/start-master.sh
# 启动当前机器的worker
sbin/start-worker.sh

# 停止全部
sbin/stop-all.sh

# 停止当前机器的master
sbin/stop-master.sh

# 停止当前机器的worker
sbin/stop-worker.sh

bin/pyspark --master spark://node1:7077
# 通过--master选项来连接到 StandAlone集群
# 如果不写--master选项, 默认是local模式运行
# 在虚拟环境内安装包
pip install pyhive pyspark jieba -i https://pypi.tuna.tsinghua.edu.cn/simple 

conda activate pyspark

pip install jieba -i https://pypi.tuna.tsinghua.edu.cn/simple 

# 启动server2
sbin/start-thriftserver.sh --hiveconf hive.server2.thrift.port=10000 --hiveconf hive.server2.thrift.bind.host=node1 --master local[*]


sbin/start-thriftserver.sh \
--name sparksql-thrift-server \
--master yarn \
--deploy-mode client \
--driver-memory 1g \
--hiveconf hive.server2.thrift.http.port=10001 \
--hiveconf hive.server2.thrift.bind.host=node1 \
--num-executors 3 \
--executor-memory 1g \
--conf spark.sql.shuffle.partitions=2
```

