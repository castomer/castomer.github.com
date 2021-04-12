---
categories: null
tags: null
title: golang server graceful shutdown
url: /2016/12/18/golang-server-graceful-shutdown/
date: 2016-12-18
author: "Dongfang Qu"
---


实现go语言serverd的优雅退出。

-------------


```golang
package main

// demo.go
// qudongfang@gmail.com

import (
	log "github.com/cihub/seelog"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func addShutDownHook(functions ...func()) {
	signals := make(chan os.Signal, 1)
	signal.Notify(signals, syscall.SIGINT, syscall.SIGTERM, syscall.SIGQUIT)
	go func() {
		for {
			s := <-signals
			switch s {
			case syscall.SIGINT, syscall.SIGTERM, syscall.SIGQUIT:
				log.Trace("got shutdown signal. ", s)

				for _, f := range functions {
					f()
				}

				log.Trace("shutdown now")
				log.Flush()

				os.Exit(0)
			}
		}
	}()
}

func main() {
	// TODO add your code here

	// add shutdown hooks
	addShutDownHook(func() {
		log.Info("exec shutdown hook: begin")

		// TODO add your code here

		time.Sleep(time.Second * 3)

		log.Info("exit now.")

		log.Flush()
	})

	log.Info("program has started successfully.")

	select {}
}

```