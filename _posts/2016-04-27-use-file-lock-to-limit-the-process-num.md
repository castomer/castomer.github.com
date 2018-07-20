---
layout: single
title: "使用文件锁防止进程被启多个"
categories: ops golang
tags: ops golang


---

基本原理：进程启动时尝试锁住自己的二进制，启动脚本通过flock命令检查二进制是否被锁。

---


```golang
package main

// demo.go
// qudongfang@gmail.com

import (
	"os"
	"syscall"
	"time"
	log "github.com/cihub/seelog"
)

func main() {
	path, err := os.Readlink("/proc/self/exe")
	if nil != err {
		return
	}

	// file lock
	lockProgram, err := os.Open(path)
	if nil != err {
		log.Criticalf("can not open file. path = %v, err = %v", path, err)
		log.Flush()
		time.Sleep(time.Second)

		os.Exit(1)
	}
	defer lockProgram.Close()

	err = syscall.Flock(int(lockProgram.Fd()), syscall.LOCK_EX | syscall.LOCK_NB)
	if nil != err {
		log.Critical("is already running. ", err)
		log.Flush()
		time.Sleep(time.Second)

		os.Exit(1)
	}
	defer syscall.Flock(int(lockProgram.Fd()), syscall.LOCK_UN)

	// TODO add your code here
	
	log.Info("program has started successfully.")

	select {}
}


```

------

```bash
# check running

# 1 = running
# 0 = not running
function check_running(){
    # TODO set/get EXEC_PATH
    flock -x -n ${EXE_PATH} -c ""

    return $?
}

```