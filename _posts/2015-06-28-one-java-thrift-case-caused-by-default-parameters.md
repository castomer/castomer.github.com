---
layout: single
title: "一个由thrift 默认参数引发的血案"
categories: java thrift case
tags: java thrift case


---

公司引入 thrift 做后端 server 之间的 rpc 框架，一个rpc sever 模块在阿里云部署的两台机器稳定运行两天后在流量低峰期诡异的退出了。

### thrift server 类型与 阿里云 ECS 配置

1. TThreadedSelectorServer
1. CPU2核、内存2GB

### server 退出前的异常log信息：

```
[2015-06-20 10:45:56,713 WARN] Got an IOException in internalRead!
java.io.IOException: Connection reset by peer
        at sun.nio.ch.FileDispatcherImpl.read0(Native Method)
        at sun.nio.ch.SocketDispatcher.read(SocketDispatcher.java:39)
        at sun.nio.ch.IOUtil.readIntoNativeBuffer(IOUtil.java:223)
        at sun.nio.ch.IOUtil.read(IOUtil.java:197)
        at sun.nio.ch.SocketChannelImpl.read(SocketChannelImpl.java:380)
        at org.apache.thrift.transport.TNonblockingSocket.read(TNonblockingSocket.java:142)
        at org.apache.thrift.server.AbstractNonblockingServer$FrameBuffer.internalRead(AbstractNonblockingServer.java:539)
        at org.apache.thrift.server.AbstractNonblockingServer$FrameBuffer.read(AbstractNonblockingServer.java:338)
        at org.apache.thrift.server.AbstractNonblockingServer$AbstractSelectThread.handleRead(AbstractNonblockingServer.java:203)
        at org.apache.thrift.server.TThreadedSelectorServer$SelectorThread.select(TThreadedSelectorServer.java:590)
        at org.apache.thrift.server.TThreadedSelectorServer$SelectorThread.run(TThreadedSelectorServer.java:545)
[2015-06-20 10:46:08,110 ERROR] run() exiting due to uncaught error
java.lang.OutOfMemoryError: Java heap space
        at java.nio.HeapByteBuffer.<init>(HeapByteBuffer.java:57)
        at java.nio.ByteBuffer.allocate(ByteBuffer.java:335)
        at org.apache.thrift.server.AbstractNonblockingServer$FrameBuffer.read(AbstractNonblockingServer.java:371)
        at org.apache.thrift.server.AbstractNonblockingServer$AbstractSelectThread.handleRead(AbstractNonblockingServer.java:203)
        at org.apache.thrift.server.TThreadedSelectorServer$SelectorThread.select(TThreadedSelectorServer.java:590)
        at org.apache.thrift.server.TThreadedSelectorServer$SelectorThread.run(TThreadedSelectorServer.java:545)

```

### 一个thrift rpc 请求接收过程如下：

参考中文注释

```
    /**
     * Give this FrameBuffer a chance to read. The selector loop should have
     * received a read event for this FrameBuffer.
     * 
     * @return true if the connection should live on, false if it should be
     *         closed
     */
    public boolean read() {
      if (state_ == FrameBufferState.READING_FRAME_SIZE) {
        // try to read the frame size completely 
		// 读取 frame 头，即frame 的size，根据 thrift 协议定义，应该是请求体前4个字节
        if (!internalRead()) { 
          return false;
        }

        // if the frame size has been read completely, then prepare to read the
        // actual frame.
        if (buffer_.remaining() == 0) {
          // pull out the frame size as an integer.
          int frameSize = buffer_.getInt(0);
          if (frameSize <= 0) {
            LOGGER.error("Read an invalid frame size of " + frameSize
                + ". Are you using TFramedTransport on the client side?");
            return false;
          }

          // if this frame will always be too large for this server, log the
          // error and close the connection.
		  // frame 长度异常判断
          if (frameSize > MAX_READ_BUFFER_BYTES) {   
            LOGGER.error("Read a frame size of " + frameSize
                + ", which is bigger than the maximum allowable buffer size for ALL connections.");
            return false;
          }

          // if this frame will push us over the memory limit, then return.
          // with luck, more memory will free up the next time around.
		   // frame 长度异常判断
          if (readBufferBytesAllocated.get() + frameSize > MAX_READ_BUFFER_BYTES) {
            return true; 
          }

          // increment the amount of memory allocated to read buffers
          readBufferBytesAllocated.addAndGet(frameSize + 4); 

          // reallocate the readbuffer as a frame-sized buffer
		  // 为 frame 分配足够大内存空间， 从 log 打印信息来看 server 就是在这个调用点 oom 了
          buffer_ = ByteBuffer.allocate(frameSize + 4);  
          buffer_.putInt(frameSize);

          state_ = FrameBufferState.READING_FRAME;
        } else {
          // this skips the check of READING_FRAME state below, since we can't
          // possibly go on to that state if there's data left to be read at
          // this one.
          return true;
        }
      }

      // it is possible to fall through from the READING_FRAME_SIZE section
      // to READING_FRAME if there's already some frame data available once
      // READING_FRAME_SIZE is complete.

      if (state_ == FrameBufferState.READING_FRAME) {
		// 读取 frame 体
        if (!internalRead()) {
          return false; 
        }

        // since we're already in the select loop here for sure, we can just
        // modify our selection key directly.
        if (buffer_.remaining() == 0) {
          // get rid of the read select interests
          selectionKey_.interestOps(0);
          state_ = FrameBufferState.READ_FRAME_COMPLETE;
        }

        return true;
      }

      // if we fall through to this point, then the state must be invalid.
      LOGGER.error("Read was called but state is invalid (" + state_ + ")");
      return false;
    } 
	
    /**
     * Perform a read into buffer.
     * 
     * @return true if the read succeeded, false if there was an error or the
     *         connection closed.
     */
	 // 读 SocketChannel
    private boolean internalRead() {
      try {
        if (trans_.read(buffer_) < 0) {
          return false;
        }
        return true;
      } catch (IOException e) {
        LOGGER.warn("Got an IOException in internalRead!", e);
        return false;
      }
    }
```

代码分析到这里问题就可以定位到是 frame size（MAX_READ_BUFFER_BYTES） 过大导致的异常退出。

```
  public static abstract class AbstractNonblockingServerArgs<T extends AbstractNonblockingServerArgs<T>> extends AbstractServerArgs<T> {
	  // 默认是 Long.MAX_VALUE，肯定 OOM
    public long maxReadBufferBytes = Long.MAX_VALUE;

    public AbstractNonblockingServerArgs(TNonblockingServerTransport transport) {
      super(transport);
      transportFactory(new TFramedTransport.Factory());
    }
  }
  
 /**
   * The maximum amount of memory we will allocate to client IO buffers at a
   * time. Without this limit, the server will gladly allocate client buffers
   * right into an out of memory exception, rather than waiting.
   */
   // 该注释指出了问题所在
  final long MAX_READ_BUFFER_BYTES;

  // 可以通过构造函数设置MAX_READ_BUFFER_BYTES
  public AbstractNonblockingServer(AbstractNonblockingServerArgs args) {
    super(args);
    MAX_READ_BUFFER_BYTES = args.maxReadBufferBytes;
  }
```


### 问题复现(验证)

1. echo "something" | nc 到 thrift 端口
1. telnet 到 thrift 端口，随意敲几个字符

thrift server log中打印几行错误信息后即退出。


### aliyun 服务器触发原因

- 阿里机房内网正常的端口扫描（猜测）
- 阿里机房内网其他用户的ECS 发起的内网攻击（猜测）

异常的客户端连接到 thrift server 端口，随机的发送数据导致 thrift协议解析到异常大的 frameSize。


### 解决办法

1. 使用 TFramedTransport，设置初始化函数中maxLength，默认值DEFAULT_MAX_LENGTH = 16384000;
1. 其他请务必设置AbstractNonblockingServerArgs.maxReadBufferBytes，默认值Long.MAX_VALUE;


### 参考代码


- [AbstractNonblockingServer::read](https://github.com/apache/thrift/blob/1568aef7d499153469131449ec682998598f0d3c/lib/java/src/org/apache/thrift/server/AbstractNonblockingServer.java#L335)
- [AbstractNonblockingServer::internalRead](https://github.com/apache/thrift/blob/1568aef7d499153469131449ec682998598f0d3c/lib/java/src/org/apache/thrift/server/AbstractNonblockingServer.java#L537)

- [TFramedTransport](https://github.com/apache/thrift/blob/1568aef7d499153469131449ec682998598f0d3c/lib/java/src/org/apache/thrift/transport/TFramedTransport.java#L77)