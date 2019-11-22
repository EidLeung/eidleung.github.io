---
layout:      post
classify:    "数据库分库分表"
title:       "分页查询Bug"
subtitle:    "Mybatis-Plus配合ShardingSphere分页查询Bug"
date:        2019-11-22
header-img: "img/work/digital-gb.jpg"
catalog:     true
author:      "EidLeung"
tags:
    - Mybatis-Plus
    - ShardingSphere
    - Bug
---

## 1. 问题再现
### 1.1. 出错代码
```
IPage<XXX> page = baseMapper.selectPage(new Page<>(currentPage, pageSize), queryWrapper);
```

### 1.2. 错误日志
![错误日志](/img/work/mybatis/log.png)
日志指出，改错误为，类型转换错误：将`Long`转换成`Integer`错误。
```
java.lang.ClassCastException: class java.lang.Long cannot be cast to class java.lang.Integer
```
最为奇怪的是，如果查询出的数据为空，是没有问题的。一旦数据库有数据，则报出上面的错误。

## 2. 分析
### 2.1. 可能原因
由于查询结果为空时正常，因此推测是由于数据库存储了`Long`型字段，二实体却使用了`Integer`来接受。
将实体中的字段全部改为`Long`，`Integer`，甚至字段只有一个`String`类型的表，只有能够查出数据，也会报出相应的问题。
因此排除**数据库字段类型不匹配**导致的问题。无赖，智能跟踪分析。

### 2.2. 跟踪源码
#### 2.2.1. 定位异常
发现抛出的异常的代码为`org.apache.ibatis.session.defaults.DefaultSqlSession`，并随后对异常进行了**重写**，导致无法正常定位。
```
@Override
public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
  try {
    MappedStatement ms = configuration.getMappedStatement(statement);
    return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```

#### 2.2.2.  跟踪异常
断点跟踪，发现原始异常抛出点在`org.apache.shardingsphere.core.optimize.sharding.segment.select.pagination`；
![异常](/img/work/mybatis/exception.png)
进一步查看源码：
![源码](/img/work/mybatis/source.png)
**发现的确是ShardingSphere强转`Integer`抛出的异常。**那么，问题来了，**ShardingSphere**为什么要强转`Integer`呢？

### 2.3. 原因分析
最初的错误日志很明显，是将`Long`强转`Integer`造成的。这里的`Long`和`Integer`分别又是什么数据的类型呢？很明显，`Integer`是**ShardingSphere**要求的数据类型；那么`Long`呢？通过跟踪发现：
![分析](/img/work/mybatis/analysis.png)
这里的`Long`是我们使用**Mybatis-Plus**分页`Page`传入的。

#### 2.3.1. 查看**Mybatis-Plus**源码
位置：`com.baomidou.mybatisplus.extension.plugins.pagination.Page`
```
public class Page<T> implements IPage<T> {
    /**
     * 每页显示条数，默认 10
     */
    private long size = 10;
    /**
     * 当前页
     */
    private long current = 1;
	//其他代码省略
}
```

#### 2.3.2. 查看**ShardingSphere**源码
位置：`org.apache.shardingsphere.core.optimize.sharding.segment.select.pagination.Pagination`
```
public final class Pagination {
	private final int actualOffset;
	private final Integer actualRowCount;
	//其他代码省略
}
```

#### 2.3.3. 结论
**Mybatis-Plus**分页使用的分页标记为`long`型，而**ShardingSphere**却为`int`，因此**ShardingSphere**会将数据强转成`int`，由于向下强转，导致抛出异常。

## 3. 解决办法
### 3.1. 修改**Mybatis-Plus**分页为`Integer`
*不建议*，因为**Mybatis-Plus**的`Page<T>`需要实现`IPage<T>`接口，而接口中也使用了`Long`，改动量很大。

### 3.2. 修改**ShardingSphere**分页为`Long`
*不建议*，因为**ShardingSphere**的分页同样设计到众多的代码，接口，改动量也很大。

### 3.3. 修改**ShardingSphere**强转`int`代码
**注意这里不是直接将`Long`转`int`，而是将`Object`转`int`**
直接修改强转代码为`parseInt`，改动量小，同时考虑到由于数据量达到超过`int`所能表示的范围的情况几乎不存在。即使超过了**ShardingSphere**本身也不支持。另外上层代码传入的本身就是数值类型，因此`parseInt`也不会抛出异常。
```
Integer.parseInt(parameters.get(((ParameterMarkerPaginationValueSegment) paginationValueSegment).getParameterIndex()).toString())
```