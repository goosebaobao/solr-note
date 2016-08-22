# 环境及准备工作

## 服务器

这里我准备了 4 台服务器，分别是

| 主机 | 用途 | ip |
|--|--|--|
| zk | ZooKeeper 3.4.8 | 172.16.82.203 |
| sc76 | Solr 5.5.2 | 172.17.21.76 |
| sc77 | Solr 5.5.2 | 172.17.21.77 |
| sc78 | Solr 5.5.2 | 172.17.21.78 |

`ZooKeeper` 和 `Solr` 的安装下载这里不再赘述，其安装目录分别为 `/data/zookeeper` 和 `/data/solr`

> **关于ZooKeeper**
> 
> 如果是生产环境，肯定会使用 `ZooKeeper` 集群，也就是最少 3 个 `ZooKeeper` 实例。这里因为是个学习环境，就使用单个 `ZooKeeper` 来搭建 `SolrCloud`
> 
> 无论是 `ZooKeeper` 集群环境的搭建，还是将 `SolrCloud` 从单个 `ZooKeeper` 实例迁移到 `ZooKeeper` 集群都是很简单的，不再赘述 

## 设定主机名

为了方便后续操作，在每台服务器的 /etc/hosts 文件里添加如下内容

```bash
172.16.82.203 zk
172.17.21.76 sc76
172.17.21.77 sc77
172.17.21.78 sc78
```

同时，修改各服务器的 /etc/sysconfig/network 文件，将 HOSTNAME 修改为相应的主机名，示例如下

```bash
NETWORKING=yes
HOSTNAME=zk
GATEWAY=172.17.21.1
```

## 关闭防火墙

执行如下命令，关闭防火墙

```bash
chkconfig iptables off
```

## 使设置生效

最后，执行 reboot 命令重启服务器

## ZooKeeper 初始状态

各服务器完成重启后，先登录到 `zk` 上，拉起 `ZooKeeper` 服务

```bash
[root@zk ~]# /data/zookeeper/bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /data/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

然后执行 `zkCli.sh` 连接到 `ZooKeeper`，并执行 `ls` 命令，可以看到当前 `ZooKeeper` 上只有一个 `znode`

```bash
[zk: localhost:2181(CONNECTED) 0] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 1] ls /zookeeper
[quota]
[zk: localhost:2181(CONNECTED) 2] ls /zookeeper/quota
[]
```

