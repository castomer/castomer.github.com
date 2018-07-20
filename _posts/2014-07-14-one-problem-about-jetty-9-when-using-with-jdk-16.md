---
layout: single
title: "jetty 9运行在jdk1.6时假死问题"
categories: java jetty jdk
tags: jetty jdk

---

## 代码

    package com.baidu.oped.aqueducts.server.status;
    
    import org.eclipse.jetty.server.Server;
    
    import org.apache.logging.log4j.LogManager;
    import org.apache.logging.log4j.Logger;
    import org.eclipse.jetty.server.nio.SelectChannelConnector;
    
    import com.baidu.oped.aqueducts.rpc.MetricsHandler;
    import com.baidu.oped.aqueducts.server.status.servlet.RpcServerStatusServlet;
    
    /**
     * http interface to get rpc server status
     *
     * @author qudongfang
     * @since JDK1.6
     */
    
    public class JettyServer {
    
        private static final Logger LOG = LogManager.getLogger(JettyServer.class.getName());
        private MetricsHandler metricsHandler;
        private int port;
    
        public JettyServer(MetricsHandler m, int p) {
            this.metricsHandler = m;
            this.port = p;
        }
    
        public void start() {
            Server server = new Server();
    
            // Set the connector
            SelectChannelConnector connector = new SelectChannelConnector();
            connector.setPort(this.port);
            server.addConnector(connector);
    
            // Set a handler
            server.setHandler(new RpcServerStatusServlet(this.metricsHandler));
    
            // Start the server
            try {
                LOG.info("starting jetty server ...");
                server.start();
                LOG.info("jetty server started.");
            } catch (Exception e) {
                LOG.error("jetty server start failed.");
                System.exit(-1);
            }
        }
    }


## 现象

编译通过，运行时无异常，代码卡死在server.start()，太TMD坑爹了。。。

## 解决方法

切换低版本的jetty。

## 后记

1. 自己太傻比，贪新版本，没仔细看release notes。
1. 你问我为啥还要使用jdk 1.6？坑爹的狼长线上服务器操作系统版本太低呀。。。

