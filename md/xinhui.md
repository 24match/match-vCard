### 技术选型？

## Spring cloud

+ 负载均衡：去除了ribbon，只需要使用spring cloud的loadbalance + resttemplate进行负载均衡

## Spring cloud gateway

+ 接口的限流（原来是使用redis对接口进行限流），redis接口防止表单重复提交
+ 解决跨域的问题
+ 需要解决一些细粒度的鉴权行为，区分前台用户以及后台用户，商户的审批流，后台商户的权限，静态资源比较少不需要使用外部nginx进行转发
  + 主要角色：超级管理员 + 商家管理员 + 店铺的前后台使用 + 前台消费者
+ 生产不停机：对流量进行管控，实现金丝雀发布（灰度发布）
+ 对接口进行熔断处理，熔断后进行转发
+ 用户的鉴权：从cookie中获取token，用token验证用户是否已经登陆，如果没有登陆则重定向到登陆页面
+ 灰度发布：如何解决需要指定配置不同版本的用户：使用断言进行配置路由的规则，一个路由可以有多个断言

## Nacos

使用nacos集群主从搭建，防止宕机，一个主机一个备机

+ 采用Raft算法，比较容易的选举算法，防止主服务器宕机

## 分布式锁

### 为什么需要分布式锁？

多个系统之间的数据需要保证正确

+ 互斥性：在任何时刻，只有一个客户端持有锁
+ 不会发生死锁
+ 加锁和解锁必须是同一个客户端，不能误解锁
+ 具有容错性

分布式锁解决方案：Redisson

+ 使用redis命令（方案最轻，性能最好，分两步执行，非原子性操作，如果加锁成功但设置过期时间失败，则会造成死锁，delete有可能误删）
  + 加锁使用`setnx`，如果成功执行之后再添加过期时间
  + 解锁使用`delete`
  + 基于redis Lua脚本：
      + 加锁：执行SET lock_name random_value EX seconds NX 命令
  + 解锁：执行LUA脚本

 ```lua
 -- 解锁：执行Lua脚本，释放锁时验证random_value 
 -- ARGV[1]为random_value,  KEYS[1]为lock_name
 if redis.call("get", KEYS[1]) == ARGV[1] then
     return redis.call("del",KEYS[1])
 else
     return 0
 end
 ```

## feign

+ 默认的feign是基于java的Urlconnect来进行请求，对每个地址保持一个长连接，需要配置httpclient + okhttp使用连接池来替换feign默认的client
+ @FeignClient：springboot会扫描有注解的类，将其注册为一个服务
+ fegin集成了ribbon，所以只有使用feign就已经具有负载均衡的作用

## 审批流选型

**Activiti**： activiti7可以使用bpnm-js流程设计器

**Camunda**：

**Flowable**：