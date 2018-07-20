---
layout: single
title: "使用简单shell实现服务进程的守护"
categories: bash ops
tags: bash ops


---

有这样一种需求：假如服务(模块)进程异常退出了，自动重启它。

社区有许多成熟的方案，常用的有以下几个：

- [daemontools](https://cr.yp.to/daemontools.html) 老牌supervise，c/c++社区
- [monit](https://mmonit.com/monit/) ruby社区用的多，附带比较多监控功能
- [commons-daemon](http://commons.apache.org/proper/commons-daemon/) java社区常用
- [akuma](http://akuma.kohsuke.org/) java社区小众lib

---

虽然以上方案都不算复杂，我还是想使用更简单的方式把这个问题解决了。

下面是从之前同事那里抄过来了一个简单方案：

```
#!/bin/bash
# control
# qudongfang@gmail.com

WORKSPACE=$(cd $(dirname $0)/../; pwd)
cd ${WORKSPACE}

readonly app=server

readonly conf="conf/${app}.conf"
readonly logPath="log"
readonly logfile="${logPath}/stdout.log"

readonly ULIMIT_RESIDENT_SET_SIZE=1000000
readonly ULIMIT_ADDRESS_SPACE=4000000
readonly ULIMIT_CORE_FILE_SIZE=1000000
readonly ULIMIT_FD_SIZE=1024

readonly SUPERVISE_NAME="supervise.${app}"

# 1 = running
# 0 = not running
function check_running(){
    # 使用文件所检测进程是否已经启动
    # 此处将来单独写篇文章来介绍
    flock -x -n ${WORKSPACE}/${conf} -c ""

    return $?
}

function start(){
    cd ${WORKSPACE}

    check_running
    if [ $? -gt 0 ];then
        echo "${app} now is running already."
        return 1
    fi

    if ! [ -f ${WORKSPACE}/${conf} ];then
        echo "Config file ${conf} doesn't exist"
        return 2
    fi

    ulimit -m ${ULIMIT_RESIDENT_SET_SIZE}
    ulimit -v ${ULIMIT_ADDRESS_SPACE}
    ulimit -c ${ULIMIT_CORE_FILE_SIZE}
    ulimit -n ${ULIMIT_FD_SIZE}

    mkdir -p ${logPath}
    mv ${logfile} ${logfile}.old 2>&1 &>/dev/null

    #start program now
    pkill -9 -f ${SUPERVISE_NAME}

    # 调用supervise 功能
    (setsid ./bin/${SUPERVISE_NAME} >/dev/null 2>&1 &)
    sleep 3s

    check_running
    if [ $? -gt 0 ];then
        pid=`pgrep -f "${app}" | sort -nr | head -n1`

        echo "${app} started. pid = ${pid}"
    else
        echo "${app} failed to start."
        pkill -9 -f ${SUPERVISE_NAME}

        return 1
    fi
}

function stop(){
    pkill -9 -f ${SUPERVISE_NAME}

    pkill -f "${app}" > /dev/null 2>&1
    sleep 3s

    for i in {1..5}
    do
        check_running
        if [ $? -gt 0 ]; then
            sleep 1s
            continue
        fi
    done

    check_running
    if [ $? -eq 0 ];then
        echo "${app} stopped."
        return 0
    else
        echo "${app} failed to stop."
        return 1
    fi
}

function restart(){
    shutdown
    start
}

function shutdown(){
    pkill -9 -f ${SUPERVISE_NAME}

    pkill -9 -f "${app}" > /dev/null 2>&1
    sleep 3s

    check_running
    if [ $? -eq 0 ];then
        echo "${app} shutdown."
        return 0
    else
        echo "${app} failed to shutdown."
        return 1
    fi
}

# for ifrit use
# no pids printed
function status(){
    check_running
    if [ $? -gt 0 ];then
        return 0
    else
        return 1
    fi
}

# for manual use
function check(){
    check_running
    if [ $? -gt 0 ];then
        pid=`pgrep -f "${app}"`

        echo "running, pid = ${pid}"
        return 0
    else
        echo "not running"
        return 1
    fi
}

function help(){
    echo "$0 start|stop|restart|shutdown|status|check"
}

function main() {
    case $1 in
        start|stop|restart|shutdown|status|check|help)
            $1
            exit $?
            ;;
        *)
            help
            exit 1
            ;;
    esac
}

main $@

```

```bash
#!/bin/bash
# supervise.server
# qudongfang@gmail.com

PATH=/usr/sbin:/sbin:/usr/bin:/bin
IFS=

readonly logfile="./log/stdout.log"

NICE_PREFIX=""
type nice >/dev/null && NICE_PREFIX="nice -n 19"

run() {
    while true
    do
        # 构造启动命令行
        cmd="${NICE_PREFIX} ./bin/server &>> ${logfile} 2>&1"

        # 记录启动命令
        echo ${cmd} >> ${logfile}

        # 执行启动
        eval "${cmd}"

        # sleep 10s 后重启
        sleep 10s

        # 记录重启时间
        echo "`date`: server restarted" >> ${logfile}
    done
}

if [ $# -eq 1 ]; then
    if [ x"${1}" = x"--run" ]; then
        run
    fi
fi

exec setsid "${0}" --run </dev/null >/dev/null 2>&1
exit 1

```

