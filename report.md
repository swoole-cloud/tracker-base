# 上报数据
[TOC]

>[success] 上报成功后在后台查看，需要切换当前项目，默认在默认项目中

![image.png](images/1563785868630-ef28ef9d-2d60-4356-ac35-a232c86a003c.png)

## php-fpm模式

如果你只使用的php-fpm模式，那么到此为止服务端后台应该在发生请求后已经有数据了

fpm下会自动创建应用，名称为域名或对应ip，可以登录服务端后台查看应用追踪等数据

## 常驻进程Service模式

目前常驻进程的业务，需要手动进行埋点，即需要在代码中添加相关代码

### 1. 创建应用

自动创建以下代码中的`$serviceName`同名应用，无需手动创建。

首先在服务端后台创建应用，在服务端->系统管理->相应项目->应用管理->新增应用，应用名即为您要监控的服务名。

> 例如：您想监控服务名为`user_service`的cli常驻进程应用，您的应用类型选择Service，服务名填`user_service`

### 2. 添加代码

```php
<?php
/**
 * 请求服务前执行
 * @param  $func eg.  App\Login\Weibo::login'
 * @param  $serviceName 必须和创建应用时候服务名一致 eg. 'user'
 * @param  $serverIp eg. '192.1.1.1'
 * @return StatsCenter_Tick object
 */
$tick = \StatsCenter::beforeReqRpc($func, $serviceName, $serverIp);
  
/*
 * 请求服务后执行
 * @param $tick  StatsCenter_Tick object
 * @param $ret   true/false
 * @param $errno 201
 * @return void
 */
\StatsCenter::afterReqRpc($tick, $ret, $errno);

 /**
 * 被调用开始前执行
 * @param  $func eg.  App\Login\Weibo::login'
 * @param  $serviceName 必须和创建应用时候服务名一致 eg. 'user'
 * @param  $serverIp eg. '192.1.1.1'
 * @return StatsCenter_Tick object
 */
$tick = \StatsCenter::beforeExecRpc($func, $serviceName, $serverIp);

/*
 * 被调用结束后执行
 * @param $tick  StatsCenter_Tick object
 * @param $ret   true/false
 * @param $errno 201
 * @return void
 */
 \StatsCenter::afterExecRpc($tick, $ret, $errno);
```

### 实例说明：

`A`请求或调用`B`时，在`A`请求服务`B`的代码前在加上`beforeReqRpc()`方法，结束时加上`afterReqRpc()`方法，此时后台可以统计到`A`调`B`的`这次`调用的成功、失败、耗时等信息，这组函数是站在调用端的角度的。

微服务框架一般都有统一的服务入口，在`B`的服务入口处(开始)加上`beforeExecRpc()`方法，服务出口处(结束)加上`afterExecRpc()`方法。此时后台可以统计到`B`被调用的所有链路信息，例如这次调用的 `mysql` `redis` 等调用都会在`afterExecRpc`后上报，这组函数是站在被调用端角度的。

## 透传TraceId/SpanId

```
<?php
/**
 * 被调用开始前执行
 * @param  $func eg.  App\Login\Weibo::login'
 * @param  $serviceName 必须和创建应用时候服务名一致 eg. 'user'
 * @param  $serverIp eg. '192.1.1.1'
 * @param string $traceId
 * @param string $spanId
 * @return StatsCenter_Tick object
 */
$tick = \StatsCenter::beforeExecRpc($func, $serviceName, $serverIp, $traceId, $spanId);

/**
 * 被调用结束后执行
 * @param $tick  StatsCenter_Tick object
 * @param $ret   true/false
 * @param $errno 201
 * @return void
 */
 \StatsCenter::afterExecRpc($tick, $ret, $errno);
```