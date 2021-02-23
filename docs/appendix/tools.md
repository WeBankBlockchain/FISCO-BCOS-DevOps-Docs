
# 脚本工具

<span id="check_seal_shell" />

## 检查节点是否共识脚本

* 创建 `~/fisco/nodes/127.0.0.1/check_seal.sh` 脚本
    * 脚本存放在跟 `nodeX` 同一级目录

```Bash
#!/bin/bash
dirpath="$(cd "$(dirname "$0")" && pwd)"
cd $dirpath

# info message
info() {
        local timestamp=$(date "+%Y-%m-%d %H:%M:%S")
	echo -e "\033[32m[INFO] [${timestamp}] $1\033[0m"
}

# error message
error() {
        local timestamp=$(date "+%Y-%m-%d %H:%M:%S")
	echo -e "\033[31m[ERROR] [${timestamp}] $1\033[0m"
}

# alarm function
alarm() {
        local message="$1"
        error "${message}"
}


# check that the node is working properly
function check_node_work_properly() {
        # node dir
        local nodedir=$1
        # node name
        local node=$(basename $nodedir)
        # fisco-bcos path
        local fiscopath=${nodedir}/../fisco-bcos
        # config.ini for this node
        local config=$1/config.ini
        # rpc url
        local config_ip="127.0.0.1"
        local config_port=$(cat $config | grep 'jsonrpc_listen_port' | egrep -o "[0-9]+")

        # check if process id exist
        local pid=$(ps aux | grep "$fiscopath" | egrep "fisco-bcos" | grep -v "grep" | awk -F " " '{print $2}')
        [ -z "${pid}" ] && {
                alarm " ERROR! $config_ip:$config_port $node does not exist "
                return 1
        }

        # get group_id list
        local groups=$(ls ${nodedir}/conf/group*genesis | egrep -o "group.*.genesis" | egrep -o "[0-9]*" | tr "\n" " ")
        for group in ${groups}; do
                # get blocknumber
                heightresult=$(curl -s "http://$config_ip:$config_port" -X POST --data "{\"jsonrpc\":\"2.0\",\"method\":\"getBlockNumber\",\"params\":[${group}],\"id\":67}")
                # echo $heightresult
                height=$(echo $heightresult | awk -F'"' '{if($2=="id" && $4=="jsonrpc" && $8=="result") {print $10}}')
                [[ -z "$height" ]] && {
                        alarm " ERROR! Cannot connect to $config_ip:$config_port $node:[group:$group] "
                        continue
                }

                height_file="$nodedir/$node_$group.height"
                prev_height=0
                [ -f $height_file ] && prev_height=$(cat $height_file)
                heightvalue=$(printf "%d\n" "$height")
                prev_heightvalue=$(printf "%d\n" "$prev_height")

                # get pbft view
                viewresult=$(curl -s "http://$config_ip:$config_port" -X POST --data "{\"jsonrpc\":\"2.0\",\"method\":\"getPbftView\",\"params\":[$group],\"id\":68}")
                # echo $viewresult
                view=$(echo $viewresult | awk -F'"' '{if($2=="id" && $4=="jsonrpc" && $8=="result") {print $10}}')
                [[ -z "$view" ]] && {
                        alarm " ERROR! Cannot connect to $config_ip:$config_port $node:[group:$group] "
                        continue
                }

                view_file="$nodedir/$node_$group.view"
                prev_view=0
                [ -f $view_file ] && prev_view=$(cat $view_file)
                viewvalue=$(printf "%d\n" "$view")
                prev_viewvalue=$(printf "%d\n" "$prev_view")

                # check if blocknumber of pbft view already change, if all of them is the same with before, the node may not work well.
                [ $heightvalue -eq $prev_heightvalue ] && [ $viewvalue -eq $prev_viewvalue ] && {
                        alarm " ERROR! $config_ip:$config_port $node:[group:$group] is not working properly: height $height and view $view no change"
                        continue
                }

                echo $height >$height_file
                echo $view >$view_file
                info " OK! $config_ip:$config_port $node:[group:$group] is working properly: height $height view $view"

        done

        return 0
}


# check all node of this server, if all node work well.
function check_all_node_work_properly() {
        local workplace=$1
        for configfile in $(ls ${workplace}/node*/config.ini); do
                check_node_work_properly $(dirname $configfile)
        done
}

function help() {
        echo "Usage:"
        echo "Optional:"
        echo "    -d  <working directory path>     default: \$dirpath/ "
        echo "    -h                  Help."
        echo "Example:"
        echo "    bash check_seal.sh -d /data/app/127.0.0.1 "
        exit 0
}

work_dir=${dirpath}

while getopts "d:f:h" option; do
        case $option in
        d) work_dir=$OPTARG ;;
        h) help ;;
        esac
done

[ ! -d ${work_dir} ] && {
        echo " working directory($work_dir) not exist, place check_seal.sh at the same directory with node directory. "
        exit 1
}

#
check_all_node_work_properly ${work_dir} ${logfile}
```

* 执行脚本

```Bash
$ cd ~/fisco/nodes/127.0.0.1 && bash check_seal.sh
```

