---
layout:      post
classify:    "分布式"
title:       "分布式事务"
subtitle:    "分布式事务解决方案"
date:        2020-07-13
catalog:     true
header-img: "img/work/digital-gb.jpg"
catalog:     true
author:      "EidLeung"
tags:
    - 分布式
    - 事务
---


# 一、[seata](http://seata.io/en-us/index.html)
```
1. 只适用于内部系统直接的交互
2. 适用于一致性要求较高的场景
3. 适用于一方出错，另一方必定回滚的情况
```
1. 通过undo_log记录修改前后的数据来实现事务的回滚
2. 无需实现反操作，有seata通过undo_log自动进行

- 问题  
1. 并发操作同一条记录的时候，如何实现回滚
2. ABA问题

# 二、TCC
```
1. 只适用于内部系统直接的交互
2. 适用于一致性要求较高的场景
3. 适用于一方出错，另一方必定回滚的情况
```
1. try执行失败，会执行cancel
2. confirm执行失败，会重试（在try阶段已经确认了资源满足条件）
3. cancel执行失败，会重试

- 开源实现
1. [tcc-transaction](https://github.com/changmingxie/tcc-transaction)
2. [Hmily](https://github.com/yu199195/hmily)
3. [byteTCC](https://github.com/liuyangming/ByteTCC)
4. [EasyTransaction](https://github.com/QNJR-GROUP/EasyTransaction)

- 需要注意的问题
1. 空回滚：没有进行try却要执行cancel（cancel需要判断是否已经执行了try，如果没有执行try，则不执行cancel）
2. 幂等：try、cancel、confirm都需要幂等
3. 悬挂：悬挂，cancel或confirm接口先于try执行（try执行时需要判断是否已经执行过cancel或confirm了，如果执行过，则不执行try）

# 三、可靠消息一致性
```
1. 只适用于内部系统直接的交互
2. 适用于一致性要不太高的场景
3. 适用于接收方执行时间较长，同时又不能阻塞发起方的的情况
4. 适用于优先保证发起方成功的情况：方发起方成功，而接收方失败对系统造成的影响不大的场景（接收方会重试直到成功或人工干预）
```
1. 本地事务与消息发送的原子性：发送消息在必须包含在本地事务中
2. 事务参与方与接收消息的可靠性：消息接收失败可以重复接收
3. 消息的重复消费：消息的幂等消费

## 1. 本地消息表[X-不建议使用]
1. 本地事务写消息表
2. 定时任务扫描消息表，发送尚未发送的消息

## 2. 事务消息
![RocketMQ事务消息](/img/doc/message.png)
- 问题
```
存在多个不同的消费端时如何处理
```
### 2.1. 发送端
0. 创建唯一的事务ID：之后的消息里面需要包含该ID
1. 发送事务消息（半消息）
2. 半消息发送成功，回调`RocketMQLocalTransactionListener`接口的`executeLocalTransaction`方法，执行本地事务，需要幂等（成功返回`COMMIT`，失败返回`ROLLBACK`）
3. 在网络出现问题的时候回调`RocketMQLocalTransactionListener`接口的`checkLocalTransaction`方法，检查本地事务执行的状态，需要幂等

### 2.2. 消费端
1. 实现`RocketMQListener`接口的`onMessage`方法（在发送端成功执行本地事务成功，也即`executeLocalTransaction`方法返回`COMMIT`后调用），需要幂等，失败重试

# 四、 最大努力通知
```
1. 适用于一致性要不太高的场景
2. 适用于跨系统交互的场景、也适用于内部系统交互场景
3. 适用于优先保证接收方成功的情况：在接收方成功后，发起方可以通过一定的补偿机制（接收消息/主动查询）确保自己也能够成功，通常在发起方已经保存了数据，只是数据状态为不可用
```
![最大努力通知](/img/doc/notify.png)
![MQ的ACK实现最大努力通知](/img/doc/in_notify.png)
![通知系统+MQ](/img/doc/out_notify.png)
## 1. 接收通知方（发起事务方）
### 1.1. 发起事务
发起事务方异步调用事务下游的系统进行事务。

### 1.2. 接收通知
接收通知方事务的执行结果通知，需要幂等（可能接收到重复通知）

### 1.3. 查询发起通知方事务执行结果
在未收到发起通知放的通知时，调用发起通知方提供的的查询接口查询事务执行情况

## 2. 发起通知方（事务下游）
### 2.1. 异步执行事务
1. 执行本地事务
2. 将本地事务的执行结果通知接收通知方

### 2.2. 查询接口（查询接口天然幂等）
提供本地事务执行成功与否的查询接口（在通知接收通知方失败后，供接收通知方主动查询结果）

# 五、其他
2PC、3PC等都是用在需要强一致性的场景，而且需要数据库支持XA协议。
