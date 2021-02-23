# 依赖安装

<span id="fisco_bcos_install_dependency" />

## 安装 FISCO-BCOS 依赖
* FISCO-BCOS 运行需要依赖 
    * `openssl`
* 部署工具依赖 
    * `wget`
    * `curl`
* JSON-RPC 依赖
    * `jq`

```Bash
# Ubuntu/Debian
sudo apt-get update && apt-get install -y openssl curl wget jq

# CentOS/RHEL
sudo yum install -y openssl curl wget jq
```

<span id="install_docker" />

## 安装 Docker 应用

如果想要使用 Docker 方式来部署节点，那么需要在主机中安装 Docker 服务。

* 安装方法

| 系统         |  版本最低要求     |
| ------------- |:-------|
|CentOS(RHEL)| CentOS 7.3（kernel >= 3.10.0-514）|
|Debian|Stretch 9|
|Ubuntu|Xenial 16.04 (LTS)|


* CentOS(RHEL)/Debian/Ubuntu

```Bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun;
```

* 关于其他系统的安装方式，请参考：[Docker 安装](https://docs.docker.com/engine/install/)

* 检查存储驱动
    * Docker 容器中的读写操作都发生在镜像层或者挂载的文件系统中，所以选择一个合适的存储驱动，有助于提高 Docker 中容器的性能和稳定性。推荐 CentOS/RHEL/Ubuntu 中都使用 `overlay2` 驱动。

```Bash
# 检查 Docker 的存储驱动是否为 overlay2
$ docker info | grep -i "Storage Driver"

# 如果输出下面结果，表示使用了 overlay2 驱动
 Storage Driver: overlay2
```

* 注意 Docker 服务的权限


<span id="install_jdk" />

## 安装 Java 运行环境
控制台（console）和 WeBASE-Front 需要依赖 Java 的运行环境 JDK。

### Ubuntu（Debian） 安装 Java

```Bash
# 安装默认Java版本(Java 8或以上)
sudo apt install -y default-jdk
# 查询Java版本
java -version 
```

<span id="centos_install_java" />

### CentOS 安装 Java

```Bash
# 查询 CentOS 原有的 Java 版本
$ rpm -qa|grep java

# 删除查询到的Java版本
$ rpm -e --nodeps java-[VERSION]

# 查询 Java 版本，没有出现版本号则删除完毕
$ java -version

# 创建新的文件夹，安装Java 8或以上的版本，将下载的jdk放在software目录
# 从 openJDK官网 (https://jdk.java.net/java-se-ri/8) 
#   或
# Oracle官网(https://www.oracle.com/technetwork/java/javase/downloads/index.html)
# 选择Java 8或以上的版本下载
# 例如下载jdk-8u201-linux-x64.tar.gz
$ mkdir /software

# 解压jdk 
$ tar -zxvf jdk-8u201-linux-x64.tar.gz

# 配置Java环境，编辑/etc/profile文件 
$ vim /etc/profile 

# 打开以后将下面三句输入到文件里面并退出
export JAVA_HOME=/software/jdk-8u201-linux-x64.tar.gz
export PATH=$JAVA_HOME/bin:$PATH 
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

# 生效profile
$ source /etc/profile 
# 查询Java版本，出现的版本是自己下载的版本，则安装成功。
java -version 
```


```eval_rst
.. admonition:: 提示

    - 需要 JDK1.8 或以上版本，推荐使用 [OpenJDK](https://openjdk.java.net/)，建议从 [OpenJDK 网站](https://openjdk.java.net/install/) 自行下载（CentOS 的 `yum` 仓库的 OpenJDK 缺少 JCE(Java Cryptography Extension) ，会导致使用该 JDK 启动的应用无法正常连接区块链节点）。
```


