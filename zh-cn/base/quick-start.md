# 快速上手

![专业辅导php](../../resource/v1n5O.jpeg)

新项目的开发可以借鉴与 [`huanhyperf/skeleton`](https://git.gaoding.com/huanhyperf/skeleton), 它是居于规范开发的快速开发模版框架，框架版本为 `2.2`

## 代码生成
`huanhyperf/devtool` 基于 `hyperf/devtool` 的基础上，扩展了其原未有的功能：
- 增加生成类型
    1. `gen:rpc-[provider|consumer]` 服务提供者 `Provider` 和消费者 `Consumer` 类
    2. `gen:service` 基础服务类
    3. `gen:[export|import]` Excel文件导入和导出类、支持格式：`XLSX`、`XLS`、`CSV`、`TSV`、`ODS`、`SLK`、`XML`、`GNUMERIC`、`HTML`
    4. `gen:test` 基于非控制器类快速生成单元测试类
    5. `gen:http-test` 基于控制器类快速生成api接口调试类
    6. `gen:doc` 基于不同的文档驱动快速生成api接口文档，支持增量和全量幂等更新
    7. `gen:env` 在无配置文件的情况下，快速生成配置文件模版
- 增加命令的可扩展性， 原有实现都是基于简单的`类模版stub` 文件快速生成，现在改为自由度更高的生成模式
- 基于模版文件快速批量生成控制器、服务类、测试类、api请求测试类


### 基于yaml或json快速生成代码
一个快速生成代码的yaml配置模版大致如下：
```yaml
Test1.Test2.Test:
    comment: '这是一个测试控制器'
    see: '具体的内容查看链接'
    const:
      SERVER_1: 1
      SERVER_2: 2
    properties:
      -
        inject: true
        type: AClass
        name: aClass
    functions:
      -
        name: queryList
        pageable: true
        comment: '查询列表'
        route: 'api/get/list'
        rules:
          - 'sinceUpdateTime|required||时间戳'
          - 'hierarchyId|required|integer||导航ID'
          - 'pageNum|nullable|integer||当前页码'
          - 'pageSize|nullable|integer||一页多少条,不能大于1000'
        messages:
          - 'pageSize.integer||页面返回条数只能为整型数字'
```

## 可穿透的上下文使用 及 协程辅助类
原框架的协程实现时，都是基于匿名函数来隔离区分不同作用域。在某一些场景下，如协程嵌套的场景下，上下文的配置则无法穿透到内部的子协程中，对于需要指定穿透的信息： 登陆态或用户上下文，需要每次指定
`skeleton` 中实现了一种可穿透配置的上下文方法， 在需要穿透的场景可以参考：
1. 在原来的协程上下文中，定义存储可穿透的上下文键名 `context:through:keys`， 要穿透配置的存储于其中
2. 改写写成调用类，统一协程的调用入口类，将需要穿透的上下文在匿名函数中传递
 ```php
class Worker
{
    public static function run(callable $func, int $limit = 0)
    {
        $parallel = new Parallel($limit);
        $through = ContextThrough::getThrough();
        $parallel->add(function () use ($func, $through) {
            foreach ($through as $key => $value) {
                ContextThrough::through($key, $value);
            }
            $func();
        });
        try {
            $res = $parallel->wait();
        } catch (ParallelExecutionException $e) {
            $res = $e->getResults();
            error($e, 'Worker Run Exception');
        }
        return $res;
    }
}
 ```

## RPC远程调用
当前欢乐逛项目的内部模块间调用， 还没有实现`rpc`相互调用。 仅有涉及中台的部分， 存在`grpc`远程调用

## 授权中间件的配置及管理

## 可追溯的TraceId机制

通过可穿透的上下文配置， 改造中间件即可以实现可以无限追溯的`TraceId` 机制

## Job异步任务的分发

以往的Job入队都是通过工厂类指定消费队列和延时之后入队， 现在通过加入可穿透配置的上下文和自分发的功能，`Job`在实例化后可以直接`dispatch` 分发，无需再在构造函数中传入用户信息

```php
public function dispatch(?int $delay = null, ?string $queue = null, ?int $max = null): bool
{
    $this->context = ContextThrough::getThrough();
    ! is_null($max) && $this->maxAttempts = $max;
    $driver = $this->getQueueDriver($queue ?? $this->bindQueue);
    return $driver->push($this, $delay ?? $this->delay ?? 0);
}

public function handle()
{
    ContextThrough::setThrough($this->context);
    try {
        $startTime = microtime(true);
        $this->execute();
        $endTime = microtime(true);
        $duration = ($endTime - $startTime);
    } catch (\Throwable $t) {

    }
}
```

## 导入和导出的定制组件
`huanhyperf/excel` 移植自 [laravel/excel](https://docs.laravel-excel.com/3.1/getting-started/)
支持但不限于以下列出的功能：
1. 支持多种格式的导入导出
2. 支持队列导入导出
3. 支持导入导出的进度条显示
4. 字段校验
5. 图表或其他高级表格导出
6. 导入到模型
7. 分片导出和分片导入


