# 运维管理

## 压力测试
对于区块链开发者和用户而言，性能是评价一个区块链平台重要的考量条件。对于 FISCO-BCOS，提供了 Caliper 来测试整个区块链服务的整体性能。

关于 Caliper 的介绍，请参考：[性能压测工具 Caliper](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/articles/4_tools/46_stresstest/caliper_stress_test_practice.html)

关于 Caliper 的使用，请参考：[Caliper 压力测试](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/caliper.html)

## 安全控制
为了保障节点间通信安全性，以及对节点数据访问的安全性，FISCO BCOS 引入了 **节点准入** 机制、**CA 黑白名单** 和 **权限控制** 三种机制，在网络和存储层面上做了严格的安全控制。

```eval_rst 
.. admonition:: 提示

    - 权限控制根据具体实现分为两种：**基于表的权限控制** 和 **基于角色的权限控制**。
```

### 节点准入
节点准入，是指基于群组的概念，将节点的接入分为两种：**网络准入** 机制和 **群组准入** 机制。每种准入机制的规则记录在配置中，节点启动后将读取配置信息实现网络及群组的准入判断，来判断是否允许节点介入区块链网络。

关于节点准入介绍，请参考：[节点准入管理介绍](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/node_management.html)

关于节点准入操作，请参考：[节点准入管理操作文档](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/node_management.html)

### CA黑白名单

**CA黑名单**

* 别称 **证书拒绝列表**（certificate blacklist，简称CBL）。CA 黑名单基于 `config.ini` 文件中 `[certificate_blacklist]` 配置的NodeID进行判断，拒绝此NodeID节点发起的连接。

**CA白名单**

* 别称 **证书接受列表**（certificate whitelist，简称CAL）。CA 白名单基于 `config.ini` 文件中 `[certificate_whitelist]` 配置的NodeID进行判断，拒绝除白名单外所有节点发起的连接。

关于 CA 黑白名单介绍，请参考：[CA 黑白名单介绍](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/design/security_control/certificate_list.html)

关于 CA 黑白名单操作，请参考：[CA 黑白名单操作手册](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/certificate_list.html)

### 基于表的权限控制
权限控制是指，基于外部账户(tx.origin)的访问机制，对包括合约部署，表的创建，表的写操作（插入、更新和删除）进行权限控制，表的 **读操作不受权限** 控制。

**权限控制规则如下：**
1. 权限控制的最小粒度为表，基于外部账户进行控制。     
2. 使用白名单机制，未配置权限的表，默认完全放开，即所有外部账户均有读写权限。    
3. 权限设置利用权限表（\_sys_table_access_）。权限表中设置表名和外部账户地址，则表明该账户对该表有读写权限，设置之外的账户对该表仅有读权限。

关于权限控制介绍，请参考：[权限控制介绍](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/design/security_control/permission_control.html#id2)
关于权限控制工具操作，请参考：[权限控制工具](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/design/security_control/permission_control.html#id9)

### 基于角色的权限控制

当前 **基于表的权限控制**，由于涉及到许多系统表，要求用户对底层的逻辑有一定理解，使用门槛较高。

从 2.5.0 版本开始，基于已有的表权限控制模型，新增了 `ChainGovernance` 预编译合约，用于实现基于角色的权限控制。

关于基于角色的权限控制介绍和使用，请参考：[基于角色的权限控制](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/design/security_control/chain_governance.html)


## 新增机构
在新增加一个机构时，需要链私钥的持有方（联盟委员会）进行操作，为新机构签发机构证书。

生成新机构文件：

* 获取机构证书生成脚本 `gen_agency_cert.sh`

* 使用脚本生成新的机构证书和私钥信息

* 生成的机构证书在 `nodes/cert/newAgencyName` 以及 `nodes/gmcert/newAgencyName-gm` 下

* 将新生成的 `newAgencyName` 中的文件发送给新机构

* **备份**生成的新机构文件

在新增机构后，如果机构需要新增节点，请参考：[节点管理](./index.html#node_management)。

```eval_rst
.. important::

    - 对新生成的机构私钥和证书 **备份**
```

<span id="group_management"/>

## 群组管理

FISCO BCOS 通过引入群组实现 **单链多账本** 功能。使联盟链从原有一链一账本的存储/执行机制扩展为一链多账本的存储/执行机制，基于群组维度实现同一条链上的数据**隔离**和**保密**。

群组间数据隔离，每个群组可以独立运行各自的共识算法，不同群组可使用不同的共识算法。

在区块链的维护操作中，常见的群组操作包括：
* 新建群组
* 启动群组
* 停止群组
* 删除群组
* 恢复群组
* 查询群组状态

当前群组管理有两种方式：

* JSON-RPC 
    * 通过 JSON-RPC 向节点发送请求，完成群组的操作
    * 具体请参考：[JSON-RPC API 接口操作](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/api.html) 中群组操作接口
* SDK 
    * 通过使用 SDK 提供的 API 接口进行编码，使用程序来操作群组
    * 使用 WeBASE-Node-Manager 服务进行群组管理，具体请参考：[群组管理](https://webasedoc.readthedocs.io/zh_CN/latest/docs/WeBASE-Console-Suit/index.html#id32)
    
```eval_rst
.. admonition:: 提示

     - 在调用群组 JSON-RPC API 接口时，请求中的 `nodeid` 参数从文件 `conf/node.nodeid` 中获取
     - WeBASE-Node-Manager 是 WeBASE 提供的一个中间件服务。依赖 WeBASE-Web，WeBASE-Front 和 WeBASE-Sign 三个服务。关于 WeBASE-Node-Manager 的部署和使用：[WeBASE-Node-Manager 节点管理服务](https://webasedoc.readthedocs.io/zh_CN/latest/docs/WeBASE-Node-Manager/index.html)
```

<span id="node_management" />

## 节点管理
FISCO-BCOS 中，一个节点代表一个 `fisco-bcos` 服务进程，在生产环境，一般一台主机只运行一条区块链的一个节点。

节点的类型分为三种类型：

* 组员
    * **共识节点**：参与共识的节点，拥有群组的所有数据（部署时默认都生成共识节点）；
    * **观察节点**：不参与共识，但能实时同步链上数据的节点；
    
* 非组员
    * **游离节点**：已启动，待等待加入群组的节点。处在一种暂时的节点状态，不能获取链上的数据；

节点管理操作包括：
* [节点加入网络](./index.html#add_node_network)
* [节点退出网络](./index.html#remove_node_network)
* [节点加入群组](./index.html#add_node_group)
* [节点退出群组](./index.html#remove_node_group)


关于节点操作的介绍文档，请参考：
* [节点组员管理和操作示例](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/node_management.html#id1)
* [节点准入管理介绍](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/design/security_control/node_management.html#id1)


<span id="add_node_network" />

### 节点加入网络
节点**加入网络**是指，新生成一个节点，并将节点加入到网络中，成为一个游离节点。

操作的主要步骤：
* 登录生成配置的主机（机构私钥所在主机）
* 下载 `gen_node_cert.sh` 脚本
* 使用机构私钥，通过 `gen_node_cert.sh` 生成新节点的配置文件
* 从其它节点拷贝 `config.ini`，`start.sh`，`stop.sh` 文件
* 修改拷贝后的 `config.ini` 文件
* 从其它节点拷贝要加入群组的 `group.x.genesis`，`group.x.ini` 文件，不需要改动
* 上传节点文件到物理主机
* 执行 start.sh 启动节点
* 检查新节点是否与原有节点创建连接
* 备份

关于节点加入网络的详细操作步骤，请参考：[节点加入网络](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/node_management.html#a)

```eval_rst
.. admonition:: 提示

     - 建议用户同时修改原有节点的 `config.ini` 文件，在 P2P 节点连接列表中加入新节点的信息并重启原有节点，保持全网节点的全互联状态。
```

<span id="add_node_group" />

### 节点加入群组

节点**加入群组**的操作，是通过控制台（console）完成的，先部署控制台（console）。

关于控制台的部署，请参考：[控制台部署和启动](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/console/console_of_java_sdk.html#id2)

操作的主要步骤：
* 节点先加入网络，请参考：[节点加入网络](./index.html#add_node_network)
* 使用控制台（console）调用 `addSealer` ，设置新节点为共识节点
* 使用控制台（console）调用 `getSealerList`，确认新节点是否在共识节点列表中
* 备份所有节点的配置文件

关于节点加入群组的详细操作步骤，请参考：[节点加入群组](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/node_management.html#id7)

```eval_rst
.. admonition:: 提示

    - 需要加入群组的群组为 `id`，就从 **该群组** 其它节点的 `config` 目录下拷贝该群组的 `group.[id].genesis`，`group.[id].ini` 配置文件
    - 节点的 `nodeid` 从节点根目录下 `conf/node.nodeid` 文件获取
    - 新节点首次启动会将配置的群组节点初始列表内容写入群组节点系统表，区块同步结束后，**群组各节点的群组节点系统表均一致**；
    - **新节点需先完成网络准入后，再执行加入群组的操作，系统将校验操作顺序；**
```

<span id="remove_node_group" />

### 节点退出群组

节点**退出群组**的操作，是通过控制台（console）完成的，先部署控制台（console）。

关于控制台的部署，请参考：[控制台部署和启动](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/console/console_of_java_sdk.html#id2)

操作的主要步骤：
* 使用控制台（console）调用 `removeNode` ，设置节点为游离节点；
* 使用控制台（console）调用 `getSealerList`，确认新节点是否在共识节点列表中
* 备份所有节点配置文件

关于节点退出群组的详细操作步骤，请参考： [节点退出群组](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/node_management.html#id8)

```eval_rst
.. admonition:: 提示

     - 节点可以以 **共识节点** 或 **观察节点** 的身份执行退出操作。
```

<span id="remove_node_network" />

### 节点退出网络
节点**退出网络**是指，将一个游离节点下线，不再接入网络。

操作的主要步骤：
* 节点先退出群组
* 修改退出的节点，清空 `config.ini` 文件中 **P2P 连接配置，** 并重启节点
* 对于同一群组的其它节点，修改 `config.ini` 文件，从 **P2P 连接配置** 中删除退出节点的信息，并重启
* 确认同一群组的其它节点和退出节点的原有连接断开
* 备份所有节点配置文件

关于节点退出网络的详细操作步骤，请参考：[节点退出网络](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/node_management.html#id6)

```eval_rst
.. important::
    - **操作节点退出网络时，必须先退出群组，退出顺序由用户保证，系统不再作校验**
```

<span id="node_upgrade" />

## 节点升级
节点升级可以通过新版本可执行文件 `fisco-bcos` 替换旧版本的可执行文件来完成，但是升级操作是仍然是属于比较危险的操作，建议在升级之前，对旧版本可执行文件进行备份。

```eval_rst
.. important::
    - 在进行升级之前，请仔细阅读 `版本升级文档和变更描述 <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/change_log/index.html>`_
    - 根据升级文档中兼容性描述，判断是否需要升级 SDK 依赖库版本。
```


### 升级方式

```eval_rst
.. important::
    - **备份，备份，备份！！** 在进行操作之前，已经要对变更文件进行备份操作！！
```

* **兼容升级**

    * 大版本需要相同，比如都是 `2.x.x`
    * 可执行文件是指节点目录中 `fisco-bcos` 文件
    * 直接使用新版本的可执行文件替换旧版本的可执行文件。比如，使用二进制为 `v2.4.0` 的可执行文件替换版本 `v2.3.x` 可执行文件 ，升级后的版本修复 `v2.3.x `中的 bug，并新增了 `v2.4.0` 动态群组生命周期管理功能、网络统计功能，但不会包含 `v2.4.0 `所有特性，普通场景下可回滚至 `v2.3.x`
    * 回滚时，使用备份的旧版本 `v2.3.x` 可执行文件替换升级的可执行文件即可

* **全面升级**
    * 参考[部署](../deploy/deploy_advise.html)部分重新搭建新链，重新向新节点提交所有历史交易，升级后节点包含 `v2.4.0` 所有新特性
    * **如果需要使用节点新特性，推荐方式为：搭建新链条，旧业务保持，新旧链共存，旧业务访问旧链，新业务访问新链；**
    * **如果不想使用新旧链共存的方式，需要重新编写业务代码，需要自行将将交易重新提交到新链上；**

```eval_rst
.. admonition:: 提示

     - 由于 `1.x.x` 和 `2.x.x` 的底层数据结果不一致，所以如果要从 `1.x` 升级到 `2.x` 不建议全面升级，推荐搭建新链的方式，保持旧链
     - FISCO BCOS 采用 `support_version` 来控制区块链兼容版本，`support_version` 在部署链的时候决定（`config.ini` 配置文件中），后续升级节点此项配置不可更改。例如建链时 `support_verison` 为 `2.1.0`，后续节点升级到 `2.2.0`、`2.3.0` 之后，`support_version` 配置必须仍然保持为 `2.1.0`。
     - 兼容升级仅包括 **大版本相同** 的版本升级，比如 `2.2.0` 到 `v2.4.0`；
```

### 升级流程
* 了解中间所有跨度版本的**更新**、**修复**、**新特性**部分，参考：[版本兼容性和变更](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/change_log/index.html)
* 确定新版本与现有 SDK，Solidity 等的兼容性
* 仔细阅读官方的升级文档，制定自己的升级方案（确定是`兼容性升级`还是`全面升级`）
* 关闭节点
* 备份现有可执行程序以及配置文件
* 替换原有可执行文件，修改配置文件
* 启动节点
* 检查节点和节点共识状态



<span id="node_move" />

## 节点迁移
节点迁移是指，当某个节点主机出现错误，或者其他原因需要更换主机时（比如云服务的主机被销毁，但是数据盘还在），将原有的节点配置文件目录和数据目录，挂载到一台新的主机，并且重启节点后，快速的完成节点变更操作。

节点迁移的大致流程：

* 挂载保存了配置文件和数据文件的磁盘到新的主机
    * 或者拷贝原有节点目录到新主机
* 修改新主机 `config.ini` 文件中 `P2P` 配置中旧的 IP 为新的 IP 地址，启动节点
* 修改其它相关节点的 `config.ini` 文件中的 `P2P` 配置，修改旧 IP 为新节点的 IP，重启相关节点
* 检查节点的共识状态

<span id="increase_disk_size" />

## 增加磁盘容量
区块链的链数据是持续增长的，对于保存共识节点和观察节点，数据会一直累积，当磁盘空间不足时，需要进行操盘扩容操作。

增加磁盘容量主要步骤：

* 如果可以，对磁盘做一个快照；
* 停止节点；
* 增加新磁盘；
* 对节点的数据目录 `data` 所在的分区进行扩容（LVM 和 非 LVM 两种方式）；
    * 有些云平台提供动态扩容，可以参考云平台的相关文档进行扩容。
* 检查分区容量是否增加；
* 启动节点；

<span id="certification_management" />

## 证书管理

证书，是指由私钥签发的 `crt` 文件，包括：
    * 链证书、
    * 机构证书
    * 节点证书、SDK 证书

在申请签发证书的时候，证书申请中会带上一个有效期的时间，一旦超过这个时间，底层节点在进行通信时，证书验证就会失败，导致无法创建网络连接，不能连接到区块链网络。
所以，证书的管理，特别是有效期检测和监控，就变得至关重要。


证书管理包括：

* [证书有效期检测](./index.html#check_certification_validation)
* [证书有效期监控](./index.html#monitor_certification_date)
* [验证证书](./index.html#verify_certification)
* [证书续期](./index.html#increase_certification_date)

在进行证书管理的操作之前，先获取证书检测脚本：

```eval_rst
.. important::
    - **当前的 `check_certificates.sh` 脚本不支持国密。**
```


<span id="check_certification_validation" />

### 证书有效期检测

关于证书有效期检测，请参考：[证书有效期检测](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/enterprise_tools/operation.html#id10)

<span id="monitor_certification_date" />

### 证书有效期监控
由于证书的重要性，所以需要随时关注证书的有效期时间。

关于证书的有效期监控，提供了两种方式：

* 使用有效期检测脚本
    * 使用证书有效期检测脚本，加上 `crontab` 定时执行，并定期发现执行结果到自定义地址

* 使用 WeBASE-Node-Manager 服务 查看
    * 部署 WeBASE-Front 服务。如果使用 WeBASE-Docker 方式部署，就不再需要单独部署 WeBASE-Front 应用
    * 使用 WeBASE-Node-Manager 服务，具体请参考：[系统管理--证书管理](https://webasedoc.readthedocs.io/zh_CN/latest/docs/WeBASE-Console-Suit/index.html#id27)
    
    

<span id="verify_certification" />

### 验证证书

关于证书验证，请参考：[证书验证](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/enterprise_tools/operation.html#id11)

<span id="increase_certification_date" />

### 证书续期
在进行证书续期前建议先检测证书的过期时间，当证书快要过期时，及时地进行证书续期操作，保证证书的有效性。

证书的续期同样需要签发方操作。链证书和机构证书都需要**联盟委员会**来进行续签操作，节点和 SDK 的证书则需要机构来进行续期操作。

* 链证书续期操作
    * 使用链私钥 `ca.key` 重新签发链证书 `ca.crt`，
    * 使用新生成的链证书 `ca.crt` 替换所有节点 `conf` 目录下的证书。

* 机构证书续期操作
    * 使用机构证书 `agency.key` 生成证书请求文件 `agency.csr`；
    * 使用链私钥 `ca.key` 对证书请求文件 `agency.csr` 签发得到机构证书`agency.crt`。

* 节点、SDK 证书续期操作
    * 使用节点 `node.key` 生成证书请求文件 `node.csr`；
    * 使用机构私钥 `agency.key` 对证书请求文件 `node.csr` 签发得到节点证书 `node.crt`；
    * 将节点证书和机构证书拼接得到 `node.crt`；
    * 使用新生成的节点证书 `node.crt` 替换所有节点 `conf` 目录下的证书。


```eval_rst
.. important::
    - 备份，备份，备份新生成的证书文件
    - 证书拼接是使用 `cat ~/myagency/agency.crt >> ./node.crt` 命令将两个文件内容拼接到一起即可
```

关于证书续期的详细操作文档，请参考：
* [节点证书续期操作](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/certificates.html?highlight=%E7%BB%AD%E6%9C%9F#id9)
* [机构证书/链证书续期](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/certificates.html?highlight=%E7%BB%AD%E6%9C%9F#id10)



<span id="data_backup_restore" />

## 数据备份与恢复
FISCO BCOS 支持两种数据备份方式，可以根据需要选择合适的方式：

* 静态备份
    * 停止节点，将节点的 data 目录整体打包压缩并备份到其他位置，待需要的时候从备份数据解压缩恢复节点。这种方式相当于快照一个账本状态的数据，以便后续从这个状态进行恢复；
    * 优点是无需部署新的服务，操作和运维简单；
    * 缺点是备份的是某个历史状态，从这个状态恢复的数据不是最新数据，恢复之后需要从其他节点同步账本更新的数据；

* 实时备份
    * 接入数据导出服务，实时将账本数据导出到链外的 MySQL，待需要恢复或新增节点时，从数据导出服务请求恢复最新的账本状态；
    * 优点是可以随时恢复到最新账本状态；
    * 缺点是需要部署服务，运维成本更高一些；



