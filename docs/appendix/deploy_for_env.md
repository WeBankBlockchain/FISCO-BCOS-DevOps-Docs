# 部署操作

<span id="build_chain_webase_docker_deploy" />

## 开发工具 和 WeBASE-Docker 部署详细操作
开发工具部署和WeBASE-Docker 部署主要依赖 `build_chain.sh` 脚本来生成节点的配置文件，区别仅仅是启动时使用的命令不一致。

* 开发工具部署
    * 使用 `bash start.sh` 启动节点
* WeBASE-Docker 部署
    * 使用 `docker run` 命令，通过 `-v` 参数，挂载配置文件的方式启动一个容器的方式启动节点

```eval_rst
.. admonition:: 提示

     - `build_chain.sh` 脚本同样可以部署生产环境，但是这种部署方式仅仅支持 `企业环境部署 <../deploy/deploy.html#product_env_deploy>`_ 中的 **单机构部署操作**。
     - 如果使用了 WeBASE-Docker 方式部署，启动的容器已经自带了 `WeBASE-Front` 应用，可以 **直接使用 WeBASE-Front 方式验证** 。
```


* **具体的部署流程：**

    * 安装部署依赖：
        * 开发者工具部署，安装 FISCO-BCOS 依赖，请参考：[安装 FISCO-BCOS 依赖](./install_software.html#fisco_bcos_install_dependency)
        * WeBASE-Docker 部署，安装 Docker 应用，请参考：[安装 Docker 应用](./install_software.html#install_docker)
    
    * 下载一键脚本
    
    ```Bash
    # 创建目录
    cd ~ && mkdir -p fisco && cd fisco
    # 下载一键脚本
    curl -LO https://raw.githubusercontent.com/FISCO-BCOS/FISCO-BCOS/master/tools/build_chain.sh
    # 脚本添加可执行权限
    chmod u+x build_chain.sh
    ```
    
    * 配置机构和群组关系
    
    ```Bash
    # 生成区块链配置文件ipconf，agency开头表示不同的机构， 1，2，3 表示群组的 id
    $ cat > ipconf << EOF
    172.16.1.101:1 agencyA 1,2,3
    172.16.1.102:1 agencyB 1,2,3
    172.16.1.103:1 agencyC 1
    172.16.1.104:1 agencyD 1
    172.16.1.105:1 agencyD 2
    172.16.1.106:1 agencyD 2
    172.16.1.107:1 agencyD 3
    172.16.1.108:1 agencyD 3
    EOF
    
    # 查看配置文件ip_list内容
    $ cat ipconf
    # 空格分隔的参数分别表示如下含义：
    # ip:num: 物理机IP以及物理机上的节点数目
    # agency_name: 机构名称
    # group_list: 节点所属的群组列表，不同群组以逗号分隔
    172.16.1.101:1 agencyA 1,2,3
    172.16.1.102:1 agencyB 1,2,3
    172.16.1.103:1 agencyC 1
    172.16.1.104:1 agencyD 1
    172.16.1.105:1 agencyD 2
    172.16.1.106:1 agencyD 2
    172.16.1.107:1 agencyD 3
    172.16.1.108:1 agencyD 3
    ```
    
    * 使用 `build_chain.sh` 脚本生成区块链节点配置文件夹
    
    ```Bash
    # 根据配置生成节点配置， 需要保证每台主机的30300~30301，20200~20201，8545~8546端口没有被占用
    # -g : 表示使用国密版本，默认使用标密。
    bash build_chain.sh -f ipconf -p 30300,20200,8545
    ```
    
    * 是否编译源码
        * 使用 WeBASE-Docker 方式部署时，可执行程序已经包含在镜像中，不能使用自编译可执行文件进行替换
        * 如果需要使用带有调试信息的可执行程序，可以通过源码编译可执行程序，然后替换掉 `nodes/172.16.1.10X/fisco-bcos `文件即可。请参考：[源码编译](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/get_executable.html#id2)
    
    * **备份**
        * 部署工具生成的所有文件中，除了 `nodes/172.16.1.10X/fisco-bcos` 可执行程序文件不用备份，备份其余的所有文件。
    
    ```Bash
    [root@host fisco]# ls -alh nodes/172.16.1.101/fisco-bcos
    -rwxr-xr-x 1 root root 23M Apr 30 10:04 nodes/172.16.1.101/fisco-bcos
    ```
    
    * 拷贝生成的配置文件到相应的物理主机
        * 如果是云主机，强烈建议将该文件夹存放在**云磁盘**中，**而不是主机磁盘**。防止因意外导致主机被销毁时，主机磁盘顺带被销毁，导致配置文件和数据丢失。
    
    ```Bash
    # 根据 IP 拷贝相应的文件夹到相应的主机
    [root@host fisco]# tree -L 1 nodes/
    nodes/
    ├── 172.16.1.101    # 拷贝该文件夹到 172.16.1.101 的主机
    ├── 172.16.1.102    # 拷贝该文件夹到 172.16.1.102 的主机
    ........
    ```
    
    * 登录每台物理主机，执行启动脚本，启动节点
        * 使用 `build_chain.sh` 部署，执行 `bash start.sh` 脚本
        * 使用 WeBASE-Docker，安装下面的格式，执行 `docker run` 命令
    
    ```Bash
    # 假设拷贝到的目录是  /opt/fisco
    scp -r nodes/172.16.1.101/* root@172.16.1.101:/opt/fisco
    ........
    ........
    # 查看拷贝后的文件
    [root@host fisco]# ls -alh /opt/fisco/
    total 21M
    drwxr-xr-x  4 root root 4.0K Apr 30 16:21 .
    drwxr-xr-x. 3 root root 4.0K Apr 30 14:48 ..
    -rwxr-xr-x  1 root root  890 Apr 30 16:21 download_console.sh
    -rwxr-xr-x  1 root root  21M Apr 30 16:21 fisco-bcos
    drwxr-xr-x  4 root root 4.0K Apr 30 16:21 node0
    drwxr-xr-x  2 root root 4.0K Apr 30 16:21 sdk
    -rwxr-xr-x  1 root root  529 Apr 30 16:21 start_all.sh
    -rwxr-xr-x  1 root root  516 Apr 30 16:21 stop_all.sh
    
    
    #### 脚本启动
    # 在 172.16.1.101 主机上，使用脚本调用命令行
    cd /opt/fisco/ && bash start_all.sh
    
    #### WeBASE-Docker 启动
    # 在 172.16.1.101 主机上，使用 Docker 方式启动，只支持国密版本
    # 1. 移动 sdk 目录，Docker 容器中的 WeBASE-Front 启动时需要使用
    cd /opt/fisco/ && mv -fv sdk node0/
    # 2. 启动容器，根据自己的需求修改镜像的 tag
    cd /opt/fisco/ && docker run -d -v /opt/fisco/node0:/data -v /opt/fisco/frontlog:/dist/log --network=host -w=/data fiscoorg/front:bsn-0.2.0-gm
    ```
    
    * 验证部署结果
        * 具体验证方式和步骤，请参考：[验证部署结果](../deploy/deploy.html#verify_deploy_result)
    
    
    相关的参考文档：
    - [开发工具多群组部署](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/group_use_cases.html#id1)
    - [开发工具使用](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/build_chain.html)
    - [WeBASE-Docker 编译和部署](https://github.com/WeBankFinTech/WeBASE-Docker/tree/dev-yhb)
   