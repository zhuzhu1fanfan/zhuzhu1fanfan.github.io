---
layout: post
title: "Hadoop 环境搭建笔记"
date: 2026-05-11
tags: [Hadoop, Linux, Java, SSH]
---

这篇文章整理 Hadoop 基础环境搭建流程，包括 JDK 安装、SSH 免密登录、Hadoop 安装、核心配置文件修改以及 HDFS 启动验证。

## 一、安装 JDK

### 1. 上传并解压 JDK

将 JDK 安装包上传到服务器后，解压到 `/opt/soft` 目录：

```bash
tar -zxf jdk1.8.0_111.tar.gz -C /opt/soft
```

### 2. 配置环境变量

编辑系统环境变量文件：

```bash
vim /etc/profile
```

在文件末尾添加：

```bash
export JAVA_HOME=/opt/soft/jdk1.8.0_111
export PATH=$PATH:$JAVA_HOME/bin
```

让配置立即生效：

```bash
source /etc/profile
```

### 3. 验证 JDK 是否安装成功

```bash
java -version
```

如果可以正常显示 Java 版本信息，说明 JDK 配置成功。

## 二、配置 SSH 免密登录

### 1. 登录本机并查看 `.ssh` 目录

```bash
ssh node01
ls -a ~/.ssh
```

### 2. 生成密钥对

```bash
ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
```

### 3. 将公钥写入授权文件

```bash
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

### 4. 测试免密登录

```bash
ssh node01
```

如果不需要输入密码即可登录，说明 SSH 免密配置成功。

## 三、安装 Hadoop

### 1. 创建安装目录

```bash
mkdir /opt/bigdata
```

### 2. 解压 Hadoop 安装包

```bash
cd /opt/soft
tar -zxf hadoop-2.7.3.tar.gz
mv hadoop-2.7.3 /opt/bigdata/
```

### 3. 配置 Hadoop 环境变量

编辑 `/etc/profile`：

```bash
vim /etc/profile
```

添加以下内容：

```bash
export HADOOP_HOME=/opt/bigdata/hadoop-2.7.3
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

让配置生效：

```bash
source /etc/profile
```

## 四、修改 Hadoop 配置文件

进入 Hadoop 配置目录：

```bash
cd $HADOOP_HOME/etc/hadoop
```

也可以使用完整路径进入：

```bash
cd /opt/bigdata/hadoop-2.7.3/etc/hadoop
```

### 1. 修改 `hadoop-env.sh`

```bash
vim hadoop-env.sh
```

将 `JAVA_HOME` 修改为：

```bash
export JAVA_HOME=/opt/soft/jdk1.8.0_111
```

### 2. 修改 `core-site.xml`

```bash
vim core-site.xml
```

添加 HDFS 默认访问地址：

```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://node01:9000</value>
  </property>
</configuration>
```

### 3. 修改 `hdfs-site.xml`

```bash
vim hdfs-site.xml
```

添加副本数、NameNode 目录和 DataNode 目录配置：

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>

  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/var/bigdata/hadoop/local/dfs/name</value>
  </property>

  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/var/bigdata/hadoop/local/dfs/data</value>
  </property>

  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>node01:50090</value>
  </property>

  <property>
    <name>dfs.namenode.checkpoint.dir</name>
    <value>/var/bigdata/hadoop/local/dfs/namesecondary</value>
  </property>
</configuration>
```

### 4. 修改 `slaves` 文件

```bash
vim slaves
```

添加 DataNode 节点：

```text
node01
```

## 五、初始化并启动 HDFS

### 1. 初始化 NameNode

```bash
hdfs namenode -format
```

> 注意：初始化命令只需要执行一次。重复执行可能会导致集群元数据不一致。

### 2. 启动 HDFS

```bash
start-dfs.sh
```

### 3. 查看 Java 进程

```bash
jps
```

正常情况下可以看到类似进程：

```text
NameNode
DataNode
SecondaryNameNode
```

## 六、配置 Windows 主机名映射

如果需要在 Windows 浏览器访问 Hadoop 页面，可以修改 hosts 文件：

```text
C:\Windows\System32\drivers\etc\hosts
```

添加节点映射：

```text
192.168.243.11 node01
192.168.243.12 node02
192.168.243.13 node03
192.168.243.14 node04
```

## 七、访问 Hadoop Web 页面

NameNode 管理页面地址：

```text
http://node01:50070
```

如果主机名无法访问，也可以使用 IP 地址：

```text
http://192.168.243.11:50070
```

## 八、常用检查命令

查看 Java 进程：

```bash
jps
```

查看 HDFS 根目录：

```bash
hdfs dfs -ls /
```

创建 HDFS 目录：

```bash
hdfs dfs -mkdir /test
```

上传文件到 HDFS：

```bash
hdfs dfs -put local.txt /test/
```

## 总结

Hadoop 环境搭建的关键步骤是：

1. 安装并配置 JDK。
2. 配置 SSH 免密登录。
3. 安装 Hadoop 并配置环境变量。
4. 修改 `core-site.xml` 和 `hdfs-site.xml`。
5. 初始化 NameNode。
6. 启动 HDFS 并通过 `jps` 和 Web 页面验证。

按照这个顺序配置，排查问题会更清晰。以后如果启动失败，可以优先检查环境变量、SSH 免密、配置文件路径和端口访问是否正常。
