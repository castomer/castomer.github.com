---
layout: single
title: "容器内外统一的taskset命令"
categories: docker ops
tags: bash taskset ops docker


---

监控业务agent部署时，为避免影响业务方计算资源，一般会对agent做cpu taskset限制。
现在遇到的问题是，agent的启动环境复杂，对于tasket来说物理机和虚拟机都很简单。然而容器就不同了，
许多种container解决方案下，容器内的进程看到的cpu都是其宿主机的cpu, taskset会失败。
为了解决这个问题，想了个笨方法先解决掉。

---

基本思路： 获取当前能看到的所有cpu核，从最后一个核尝试绑定执行一个简单的命令；如果执行成功，则认为我的agent也可以绑定成功。

----

代码大致如下：

```bash

findLastUsableCore() {
    count=`grep -c ^processor /proc/cpuinfo`

    count=$((count - 1))
    while [ "${count}" -ge "0" ] ; do
        taskset -c ${count} echo >/dev/null 2>&1

        if [ "$?" -eq "0" ];then
            return ${count}
        fi

        count=$((count - 1))
    done

    return 0
}

```



------------------

容器内为什么还要用taskset?

的确，容器就是容器，容器不应该再跑多进程，不应该把它当虚拟机用。我也很讨厌把容器当虚拟机用的现状。
解决的问题好像是本不应该出现的问题。方向都是错的。大家不要喷我为啥要要做容器内的taskset了，我司的主要业务大多跑在`sleep 9999d`启动下的容器里。。。。
