---
categories:
  - kafka
tags:
  - kafka
  - ruby
  - dashing
  - monitoring
title: 使用dashing写一个kafka监控dashboard
url: /2014/01/01/write-a-kafka-monitoring-dashboard-using-dashing-in-2-minutes/
date: 2014-01-01
author: "Dongfang Qu"
---


## problem 

---

[kafka](http://kafka.apache.org/)是一个以高吞吐著称的分布式消息队列组建，由linkedin于2010年12月开源，目前已经apache的正是项目, 2013年12月发布0.8稳定版，
官方一直没有提供比较好的状态监控解决方案，仅提供了一些[JMX的MBean接口](http://kafka.apache.org/documentation.html#monitoring)，本文将基于这些[JMX](http://zh.wikipedia.org/wiki/JMX)的接口构建一个基本的kafka核心状态监控dashboard。  

  目前我们还没有从0.72迁移到最新版本，所以本文仍以kafka 0.72为例。

---

## tools

---

1. dashing
[dashing](http://shopify.github.io/dashing/)是shopify开源的一个简单、易用又不失强大的dashboard框架，几本的使用步骤如下：
>1. `sudo gem install dashing` # 安装
>1. `dashing new kafka_dashboard` # 创建示例项目
>1. 定制dashboards布局 # dashboards目录，参考sample.erb和sampletv.erb文件
>1. 编写数据jobs # jobs目录，参考示例项目*.rb
>1. `cd kafka_dashboard && dashing start` # 启动 

1. jmxcmd
[jmxcmd](http://sourceforge.net/projects/jmxcmd/)是一个简单的jmx命令行客户端工具，当然如果你用的是jruby的话，可以直接使用[jmx gem](http://rubygems.org/gems/jmx)，
目前原生的ruby还没有可以直接访问jmx的方法，所以此处引入了这个叫jmxcmd的外部jar包。

---

## how 

---

1. 首先定义定义要监控的数据项

``` ruby
<% content_for :title do %>aqueducts dashboard<% end %>
<div class="gridster">
  <ul>
    <li data-row="1" data-col="1" data-sizex="2" data-sizey="1">
      <div data-id="bytesIn" data-view="Graph" data-title="kafka In in 3s" style="background-color:#009618"></div>
    </li>

    <li data-row="1" data-col="1" data-sizex="2" data-sizey="1">
      <div data-id="bytesOut" data-view="Graph" data-title="kafka Out in 3s" style="background-color:#ff9618"></div>
    </li>

  </ul>
</div>
```

2. 编写jobs从kafka获取监控数据

```ruby
@numOfPoints = 15
@kafkaHost = "my.kafka.host"
@kafkaPort = "9999"
@jmxcmdPath = "/home/castomer/bin/jmxcmd.jar"
@jmxBean = "kafka:type=kafka.BrokerAllTopicStat"

def getKafkaStatus(bean, key)
  value = `java -jar #{@jmxcmdPath} - #{@kafkaHost}:#{@kafkaPort} #{bean} #{key} 2>&1`
  # jmxcmd 默认把输出打印到stderr，因此需要将stderr重定向到stdout
  return value.split(": ").last.to_i;
end
  
bytesIn = []
bytesOut = []
pointsIn = []
pointsOut = []

bytesIn <<  getKafkaStatus(@jmxBean, "BytesIn")
bytesOut << getKafkaStatus(@jmxBean, "BytesOut")

(1..@numOfPoints).each do |i|
  bytesIn << getKafkaStatus(@jmxBean, "BytesIn")
  bytesOut << getKafkaStatus(@jmxBean, "BytesOut")
  
  pointsIn << { x: i, y: bytesIn[i] - bytesIn[i - 1] }
  pointsOut << { x: i, y: bytesOut[i] - bytesOut[i - 1] }
end

last_x = pointsIn.last[:x]
 
SCHEDULER.every '3s' do
  bytesIn.shift
  bytesOut.shift
  pointsIn.shift
  pointsOut.shift
  
  last_x += 1
  
  bytesIn << getKafkaStatus(@jmxBean, "BytesIn")
  bytesOut << getKafkaStatus(@jmxBean, "BytesOut")
  
  pointsIn << {x: last_x, y: bytesIn[@numOfPoints] - bytesIn[@numOfPoints - 1] }
  pointsOut << {x: last_x, y: bytesOut[@numOfPoints] - bytesOut[@numOfPoints - 1] }
  
  send_event('bytesIn', points: pointsIn)
  send_event('bytesOut', points: pointsOut)
end

```

---

## result

---

![kafka bytes in](/img/kafka_bytes_in.png?raw=true "kafka bytes in")
![kafka bytes out](/img/kafka_bytes_out.png?raw=true "kafka bytes out")

---

## todo

---

以上实现kafka throughout的dashboard监控，还有一些其他的[监控指标](http://kafka.apache.org/documentation.html#monitoring)在需要的时候再增加咯，下班回家～
