---
categories:
  - java
  - fastjson
tags:
  - java
  - fastjson
title: api接口中设置double浮点类数的最大小数位
url: /2017/04/25/how-to-set-max-decimal-size-to-n-using-fastjson/
date: 2017-04-25
author: "Dongfang Qu"
---


java开发的restful api中，偶尔需要将最大有效位限制在一个固定的范围内，如:最多保留小数点后4位。
一番dig后，整理一个简单方案记录如下:

```
SerializeConfig.getGlobalInstance().put(Double.class, new DoubleSerializer(new DecimalFormat("#.####")));
```

```java
// 测试用例
    @Test
    public void floatDataFmtTest() {
        Double[] data = new Double[]{123456789d, 1d, 1234d, 12345d, 0.1d, 0.1234d, 0.12345d};

        DecimalFormat df = new DecimalFormat("#.####");
        for (Double v : data) {
            System.out.println(df.format(v));
        }
    }
```

```bash
# 输出结果
123456789
1
1234
12345
0.1
0.1234
0.1235
```