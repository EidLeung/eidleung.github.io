---
layout:      post
classify:    "极客时间"
title:       "Netty源码剖析与实战-基础知识"
subtitle:    "如何构建生产级Netty应用"
date:        2019-11-15
catalog:     true
header-img: "img/geekbang/geekbang-main.jpg"
header-img-credit: "@geekbang.org"
header-img-credit-href: "www.geekbang.org"
author:      "EidLeung"
tags:
    - 极客时间
    - Netty
---

<center><h1><b>Netty基础知识</b></h1></center>
<p align="right"><a href="#如何购买">购买专栏</a></p>

## 1. Netty比JDK提供的NIO多做了哪些
1. 支持常用的应用层协议
2. 解决传输时的粘包、半包等问题
3. 支持流量整形
4. 提供完善的断连、Idle等异常处理
5. 规避了JDK.NIO的bug
> 1. epoll的selecter导致CPU升至100%的问题
> 2. IP_TOS参数抛出异常问题

## 2. I/O 模式
**Netty主要是使用NIO，其他I/O模式被标记为废除**
- 阻塞、非阻塞
> 是否等待
- 同步异步
> 数据就绪后，谁来读

I/O模式|同步/异步|阻塞/非阻塞
:-:|:-:|:-:
BIO（OIO）|同步|阻塞
**NIO**|同步|非阻塞
AIO（NIO2）|异步|非阻塞

1. 暴露更多的可用参数：边缘触发+水平触发
2. Netty的垃圾回收更少、性能更好

## 3. Netty如何切换I/O模式
> 利用**泛型+反射+工厂**实现I/O模式的切换，由于Netty将BIO和和AIO标记为废除，因此建议直接使用NIO。

1. 将EventLoopGroup切换成相应I/O模式的EventLoopGroup。
2. 将SocketChannel切换成相应的ocketChannel。

## 4. Netty的Reactor
- Reactor  
`注册感兴趣的事件 --> 扫描事件是否发生 --> 发生了做相应的处理`

1. Reactor单线程
```
EventLoopGroup eventGroup = new NioEventLoopGroup(1);
ServerBootstrap serverBootstrap = new ServerBootstrap();
serverBootstrap.group(eventGroup);
```
*注：* 这里的1表示线程数量，不指定会根据CPU核心数自动生成。

2. Reactor多线程
```
EventLoopGroup eventGroup = new NioEventLoopGroup();
ServerBootstrap serverBootstrap = new ServerBootstrap();
serverBootstrap.group(eventGroup);
```
3. 主从Reactor单线程  
```
EventLoopGroup bossGroup = new NioEventLoopGroup();
EventLoopGroup workerGroup = new NioEventLoopGroup();
ServerBootstrap serverBootstrap = new ServerBootstrap();
serverBootstrap.group(bossGroup, workerGroup);
```

## 5. 粘包和半包
- 原因  
发送的数据太小或太大
- 解决办法  
解码：ByteToMessageDecoder

方式|解码|编码
:-:|:-:|:-:
固定长度|FixedLengthFramDecoder|无：几乎不用
分隔符|DelimiterBasedFramDecoder|无：实现简单
固定长度+内容长度|LenghtFixedBasedFramDecoder|LenghtFixedPrepender

## 6. 二次编解码
> MessagetoMessageDecoder

`数据流 --> |1次|字节数组 --> |2次|Java对象`

## 7. Keepalive和Idle检测
1. 开启Keepalive
```
//任选其一：建议使用NioChannelOption
bootstrap.childOption(ChannelOption.SO_KEEPALIVE, true);
bootstrap.childOption(NioChannelOption.of(StandardSocketOption.SO_KEEPALIVE),true);
```
2. 开启Idle检测
```
ch.pipeline().addLast("idleCheckHandler", new IdleStateHandler(
    0/*readerIdleTime*/, 20/*writerIdleTime*/, 0/*allIdleTime*/, TimeUnit.SECONDS));
```

## 8. Netty堆外内存
### 8.1. 池化
1. 参数：`io.netty.allocator.type`
- `pooled` 使用池化
- `unpooled` 使用非池化
2. 代码：`UnPooledByteBufAllocator`或`PooledByteBufAllocator`
```
//使用非池化
bootstrap.childOption(ChannelOption.ALLOCATOR, UnPooledByteBufAllocator.DEFAULT);
//使用池化
bootstrap.childOption(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT);
```

### 8.2. 堆内/外
1. 参数：`io.netty.noPreferDirect`
- `false` 使用堆外
- `true` 使用堆内
2. 代码：`**ByteBufAllocator`参数为`true`或`false`
```
//使用堆外内存
bootstrap.childOption(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT);//默认使用堆外
//使用堆内内存
bootstrap.childOption(ChannelOption.ALLOCATOR, new PooledByteBufAllocator(false));
```

---
###### 如何购买
1. 扫描二维码
	<div align="center">
		<a href="https://time.geekbang.org/course/intro/237?code=AQfeKSBrHdDauCqprpnDXshxJSdGKJfytyRWlP0r0V0%3D">
			<img src="/img/geekbang/netty.jpg" width = "300" height = "300" alt="图片名称" style="display: inline-block"/>
		</a>
		<img src="/img/JTYK.jpg" width = "300" height = "300" alt="今天有课" style="display: inline-block"/>
	</div>
2. 点击链接  
[极客时间-直连](https://time.geekbang.org/course/intro/237?code=AQfeKSBrHdDauCqprpnDXshxJSdGKJfytyRWlP0r0V0%3D)  
[今天有课-红包](https://jika.nali.net/youke/coupon/getCouponList?sendUserId=17140)