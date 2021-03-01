# 环境准备

## 选择版本
FISCO-BCOS 每个版本的兼容列表，请参考：[ FISCO-BCOS 版本及兼容](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/change_log/index.html#id1)

<span id="disk_data_encryption" />

## 落盘加密

在联盟链的架构中，机构和机构共同搭建一条区块链，数据在联盟链的各个机构内是可见的。

在某些数据安全性要求较高的场景下，联盟内部的成员并不希望联盟之外的机构能够获取联盟链上的数据。此时，就需要对联盟链上的数据进行访问控制。主要分为两个方面：

* 链上通信数据的访问控制
    * 通过节点证书和SSL来实现

* 节点存储数据的访问控制
    * 通过落盘加密来实现。落盘加密是对节点存储在硬盘上的内容（合约的数据、节点的私钥）进行加密，就算硬盘被带出联盟链自己的内网环境，也无法解密链中的数据，保证了数据的安全

```eval_rst
.. important:: 

     - 节点在第一次运行前，必须配置好是否采用落盘加密。一旦节点开始运行，无法切换状态。
```


### 启用落盘加密特性

* 启用落盘加密特性主要有三步：
    1. 部署 Key Manager 应用;
    2. 使用 `gen_data_secure_key.sh` 生成节点的配置;
    3. 修改节点配置，添加生成的落盘加密配置;

关于**落盘加密**的详细介绍和操作文档，请参考：[落盘加密](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/design/features/storage_security.html)


## 国密版本

为了充分支持国产密码学算法，金链盟基于国产密码学标准，实现了国密加解密、签名、验签、哈希算法、国密 SSL 通信协议，并将其集成到 FISCO-BCOS 平台中，实现了对国家密码局认定的商用密码的完全支持。

国密版 FISCO-BCOS 将交易签名验签、P2P 网络连接、节点连接、数据落盘加密等底层模块的密码学算法均替换为国密算法，国密版 FISCO-BCOS 与标准版主要特性对比如下：

|   | 标准版FISCO-BCOS | 国密版FISCO-BCOS      |
|----------------------|--------------------|---------------|
| SSL链接     | Openssl TLSv1\.2 协议 | 国密 TLSv1\.1 协议  |
| 签名验证     | ECDSA 签名算法          | SM2 签名算法       |
| 消息摘要算法     | SHA\-256 SHA\-3    | SM3 消息摘要算法     |
| 落盘加密算法   | AES\-256 加密算法       | SM4 加密算法       |
| 证书模式      | OpenSSL 证书模式        | 国密双证书模式   |
| 合约编译器  | 以太坊 solidity 编译器     | 国密 solidity 编译器 |

关于**国密支持方案**，请参考：[国密支持方案](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/design/features/guomi.html)

<span id="compile-source-code" />

## 源码编译

为了压缩可执行程序的文件大小，减少下载时的网络传输时间，官方提供的可执行文件去掉了调试信息。如果需要带有调试信息的可执行程序，方便后续调试，可以自行通过编译源码获得可执行程序。

使用编译后的可执行程序替换部署工具生成的 `fisco-bcos` 可执行文件即可。


```eval_rst
.. important::

    - 
```


关于**源码编译**，请参考：[源码编译](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/get_executable.html#id2)


## 检查主机信息

如果主机是云主机，由于云主机的公网 IP 均为虚拟 IP。若生成的节点配置文件 `config.ini` 中 `rpc_ip`，`p2p_ip`，`channel_ip` 三个配置填写了外网 IP，会绑定失败，必须填写 0.0.0.0。

```eval_rst
.. admonition:: 提示

     - 关于 `config.ini` 文件的配置，请参考： `config.ini 配置文件 <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/configuration.html#config-ini>`_
```

* 网络连接
    * 确保各个节点之间是否能够相互通信（使用 `ping`，`telnet` 命令检查），如果不能访问，配置云主机的安全组，路由配置。

* 端口开放
    * 区块链底层 FISCO-BCOS 在启动的时候，会涉及到三个端口：

| ‌端口类型           | ‌默认值    | ‌描述        | ‌建议 |
|----------|---------|----------------|-----------|
| ‌p2p\_port      | ‌30300  | ‌节点之间互联，包括机构内的多个节点，<br />以及多机构节点之间的互联  | ‌绑定外网IP 或者 0\.0\.0\.0<br /><br />**注意：**云主机的公网 IP 均为虚拟 IP 或者动态 IP，只能使用 0\.0\.0\.0  |
| ‌jsonrpc\_port  | ‌8545   | ‌JSON\-RPC API HTTP 请求端口             | ‌绑定内网 IP 地址，仅供本机和内网访问   |
| ‌channel\_port  | ‌20200   | ‌控制台和客户端 SDK 连接的 Channel 端口          | ‌绑定内网 IP 地址，仅供本机和内网访问|

```eval_rst
.. admonition:: 提示

    - 如果配置 `jsonrpc_port` 只能本机访问，FISCO-BCOS 2.3.x 之后的版本（包括 2.3.x）配置 `jsonrpc_listen_ip` 为 `127.0.0.1` 即可。如果是 2.3.x 之前的版本，只能在网络防火墙屏蔽访问 `8545` 端口，请参考： `配置 RPC <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/configuration.html#rpc>`_
   
```

```eval_rst
.. admonition:: 提示

     - 如果绑定内网 IP 地址，就只能本机和内网访问。如果需要某个端口只能本机访问，设置绑定 IP 为 `127.0.0.1`；
     - 监听端口必须位于 `1024-65535` 范围内；
     - 云主机一般通过 **安全组规则** 和 **本机防火墙** 配置端口开放规则；
     - 私有主机一般是通过 **本地 NAT** 和 **本机防火墙** 配置端口开放规则；
```
