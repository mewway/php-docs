# TodoList
-----
# 2022

## huanhyperf/devtool
> devtool是代码快生成工具，在原来的hyperf/devtool的基础上 进行了深化修改， 扩展支持了多种类模版生成， 支持从json、yaml配置文件中、一键生成控制器、服务、模型、单元测试代码、支持一键生成api文档
### Features
```bash
gen:amqp-consumer   amqp 消费者类
gen:amqp-producer   amqp 生产者类
gen:aspect          快速生成切面类
gen:command         快速生成命令含
gen:constant        快速生成常量类
gen:controller      快速生成控制器
gen:dic             根据数据库生成当前项目的编程词典
gen:doc             根据控制器快速生成api文档
gen:env             快速方向根据项目config生成env配置文件
gen:event           快速生成事件触发类
gen:export          快速生成导出类
gen:http-test       快速生成http请求测试类
gen:import          快速生成导入类
gen:job             生成任务类
gen:kafka-consumer  kafka消费者类
gen:listener        生成监听类
gen:middleware      生成中间件
gen:migration       生成迁移
gen:model           生成模型
gen:nats-consumer   nats 消费者类
gen:nsq-consumer    nsq 消费者类
gen:process         Create a new process class
gen:request         生成表单请求
gen:resource        生成资源
gen:rpc-consumer    rpc 客户端请求类
gen:rpc-provider    rpc 服务端提供者类
gen:seeder          数据工厂类
gen:service         快速生成逻辑服务类
gen:test            快速生成单元测试
```
### TODO
- api 文档生成待完善
- 单元测试 的快速生成待完善

----- 
## huanhyperf/excel

> excel是通用的文件导入到处组件
对比以往的导入导出工具、主要在功能上支持参数校验、支持多格式的导入导出、 模型导入导出、模版快速下载等功能

-----

# 2023

## huanhyperf/squealer

这个组件旨在自动生成用户操作日志和系统操作日志、并支持根据不同策略触发对应操作，衍生的背景是抖音小店的多次资损
组件的主要功能点规划如下：
- 监听指定的模型、生成可读性较高的前台用户操作可追溯记录
- 记录敏感信息变化、生成系统日志
- 特定模型下、根据不同策略监听不同字段
    1. 如价格监听策略， 当监听的价格变化幅度或变化值达到指定阈值，触发响应的 钉钉消息推送或其他 风险警报