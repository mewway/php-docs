# 框架使用指南

## 框架

欢乐逛及企业电商解决方案当前的主要开发语言为: PHP, 主要使用的框架为 `Hyperf` 版本为: `2.0` `2.2`

>   [Hyperf 文档中心](https://hyperf.wiki/2.2/)

在此基础上, 内部维护了多个常用的Composer业务组件

-   huanhyperf 命名空间下, 主要囊括了hyperf框架相关的业务组件 及 各平台接入的sdk
    -   平台的sdk诸如 `huanhyperf/alibaba` `huanhyperf/taobao`
    -   数据库组件 `huanhyperf/database`
-   hlg 命名空间下, 主要包括各个平台的应用

### 框架的基础目录划分

当前框架的主要目录结构如下:

```php
├── app # 应用主目录
│   ├── Annotation # 自定义注解的目录
│   ├── Command # 自定义命令行脚本目录
│   ├── Component # 自定义库的目录
│   ├── Constants # 常量配置
│   ├── Controller # 控制器
│   ├── Crontab # 定时脚本
│   ├── Dispatcher # 任务分发
│   ├── Event # 事件
│   ├── Exception # 异常处理
│   ├── Helpers # 辅助函数
│   ├── Job # 定时任务
│   ├── Listener # 事件监听
│   ├── Middleware # 中间件
│   ├── Model # 模型
│   ├── Process # 进程
│   └── Service # 服务类
├── bin
├── config # 配置目录
│   ├── autoload # 常用配置
│   └── routers # 路由配置
├── grpc
├── migrations # 数据库迁移文件
├── proto # grpc proto协议文件目录
├── runtime # 缓存
├── storage # 静态资源
├── test # 单元测试目录
└── vendor # composer组件目录
```
目录下的文件命名规范 *必须* 参照如下：
- 目录划分了类种类或行为，因此，目录下文件的命名 *必须* 以目录所在的种类为 `suffix` 命名
  1. Controller => XXXController
  2. Service => XXXService
  3. Command => XXXCommand
  4. Exception => XXXException
  5. Job => XXXJob
  6. Process => XXXProcess
  7. Middleware => XXXMiddleware

- Command类的命名 *应该* 和命令的命名 保持一致
  ```php
    #执行的命令为：
    php bin/hyperf.php category:update-everyday
    #类名
    CategoryUpdateEverydayCommand
  ```

## 配置文件与环境变量

-   `.env` 的配置 **不应该** 被纳入版本管理
-   本地新增的 `env` 配置信息, **应该** 及时同步到 `.env.example`中, 避免因为你新增的配置导致项目运行问题
-   `env()` 除了在 `config`目录下, **不应该** 在其他地方使用, 此目录外的配置读取 **应该** 使用 `config()`
-   `config` 目录下使用 `env()` 配置的时候, **应该** 加上初始化的值, 如果该值与环境关联, **应该** 优先使用于生产环境关联的配置信息



## 辅助函数

辅助函数 **必须** 存放在位置: `app/Helpers/Helper.php` 并由`composer` autoload 自动加载引入:

```json
"autoload": {
        "files": [
            "app/Helpers/Helper.php"
        ]
    },
```

## API开发的准则

### 请求的格式和响应的格式

在常用的请求响应格式中, 主要包含以下几种:

-   Snake 小写下划线的风格, 形如: `i_love_tomato`
-   MixedCaps 驼峰风格, 其中包括大驼峰, 形如: `ILoveTomato`, 小驼峰, 形如:`iLoveTomato`

请求的入参数及路由的命名 **必须** 严格按照 **小写下划线** 的风格命名, 接口的响应也 **必须** 按照 **小写下划线** 的风格响应

> 路由的命名中下划线使用？： 1. 中划线 （seo友好） 2. 下划线（历史用法，沿用）

### 原则
- 接口 *应该* 预留版本的设计
  1. 默认版本 `v1` *应该* 默认不在路由中体现 `/api/material/:id`
  2. V2版本： `/api/v2/material/:id` 对应控制器中的方法 *应该* 以版本为后缀命名
- 接口开发 **应该** 遵循`RESTful`架构的规则, 即路由即资源, 如对投放素材的操作设计时, 路由应该设计为:

    -   GET 资源的获取, 对应路由设计为
        -   `/api/material/:id => 获取单个素材的详情`
        -   `/api/material/:id/item => 获取指定素材的所有商品`
        -   `/api/material/:id/item/:id => 获取指定素材的指定商品`
        -   `/api/material => 获取素材的列表`
    -   POST 资源的创建 , 对应路由设计为 `/api/material/:id => 创建素材`
    -   DELETE 资源的移除 , 对应路由设计为 `/api/material/:id => 移除素材`
    -   PUT 资源的更新 , 对应路由设计为 `/api/material/:id => 更新素材`

- 公用请求参数 **必须** 按照以下传入:

    -   page: 页码
    -   page_size: 分页的尺寸, 每页返回的数量
    -   order_by: 根据某一个字段排序
    -   sort: 排序的方式 asc - 升序, desc - 逆序

- 对于敏感数据, **绝不** 在接口中返回, 接口返回 **应该** 适量适当, 按需返回

- 对象节点的返回, **应该** 要有默认值

    ```json
    # 如下示例 在店铺关联管理员的时候, 如果能关联到管理员 返回对象
    {
        "shop_id": 1,
        "owner_id": 2,
        "owner_info": {
            "name": "张三"
        }
    }
    ```

    ```json
    # 在店铺关联管理员的时候, 如果关联的管理员发生异常或无法读取, 不应该返回 null 应该返回空对象(模型关联时增加默认值的返回)
    {
        "shop_id": 1,
        "owner_id": 2,
        "owner_info": {
        }
    }
    ```

- 接口响应的基本格式 **必须** 按照下列格式返回:

    ```json
    {
        "code": 10000,
        "message": "success",
        "data": []
    }
    ```

- 分页的响应基本格式, **必须** 按照下列格式返回:

    ```json
    {
        "code": 10000,
        "message": "success",
        "data": [
            {},
            {}
        ],
        "page":1,
        "page_size":20,
        "total":100
    }
    ```


### 错误码的约定

由于历史的原因, 接口的响应码有两种标准:

-   多平台项目中 `code => 10000` 标识接口正常响应
-   电商项目中 `code => 0` 标识接口正常响应

除此之外, 错误的接口响应码格式 **必须** 为 整型, **应该** 定长四位, 命名的标准 **可以** 参考如下:

- 响应码的具体意义 **应该** 如下：

    - 第一位：标识错误的来源 标识业务场景
    - 第二位：标识错误的等级及致命程度 0-9 致命程度递增
    - 第三位：同第四位 错误的随机码
    - 第四位：同第三位 错误的随机码

- 第一位错误标识的定义 **应该** 如下:

    - AUTH（authorization 授权、权限、权益相关）对应响应码开头为：1
    - VAL（validation 值验证, 表单验证） 对应响应码开头为：2
    - SYS（system 系统内部, 逻辑处理异常的错误类型）对应响应码开头为：3
    - NET（http 网络请求, 白名单、限流相关的错误类型）对应响应码开头为：4
    - CUS  (custom 自定义的错误类型) 对应响应码开头为 5
    - IDS（inner dependency service 内部服务调用报错, 如rpc、跨模块、应用间调用报错）对应响应码开头为：6
    - THR（third party 依赖第三方, 除去企业内部模块或应用的依赖第三方的请求错误）对应响应码开头为：7
    - DB（database 数据库执行的错误码）对应响应码开头为：8
    - CON (contract缩写 和前端的契约响应码  前端会根据此类响应码做特殊交互) 对应开头 9

### 接口文档的维护

  接口文档 **应该** 使用yapi维护, 并保持更新, 后续 **可以** 改用 ApiDoc 或 Swagger

  >   [Yapi 文档中心](https://yapi.gaoding.com/)

### Http状态码使用约定

接口响应码 *应该* 使用以下推荐值

- 200 请求成功
- 201 POST数据创建成功的响应码
- 202 需要异步处理的， 不能立刻返回结果的响应码
- 204 DELETE 或 PUT方法操作成功的响应码
- 301 请求资源被永久重定向
- 302 请求资源被临时重定向
- 400 请求的语法错误，服务器无法正确处理
- 401 权限校验失败，特指过期，失效
- 403 权限不足， 特指越权操作，或未经授权的操作
- 404 资源不存在
- 405 错误的请求方法
- 406 不支持的数据格式， 用户预期的返回格式无法支持
- 429 请求达到并发限制
- 500 服务器错误
- 502 网关超时
- 503 服务器维护中
-
完整的http 状态码的标准 *可以* 参照  [Http](https://baike.baidu.com/item/HTTP/243074?fr=aladdin) , 且约定如下:

### 框架的基础使用

#### Controller 、Service 与 Model

##### Controller

项目中对于控制器的定位:控制器的作用是做表单、参数验证而已, 控制器的规范如下:

-   控制器 **应该** 按照场景及其通用性来划分目录, 命名 **应该** 简短, 通用的控制器或父控制器 **应该** 放在控制器目录的根目录下, 其他的控制器 **应该** 按照业务或版本需求划分

-   控制器中 **不应该** 出现逻辑层处理的代码, 逻辑处理 及 数据处理 **应该** 统一在`Service` 层调用, 在`Query`  层 或 `Model` 层实现

-   控制器 **应该** 只包含路由动作方法 及其 参数验证的功能, 不**应该** 夹杂 业务逻辑

-   控制器中方法的命名 **应该** 优先参照 `RESTFul`架构标准, 常见接口基本都是CURD的场景, 这种场景下, 控制器的创建 **应该** 以资源的粒度进行创建

    >   对资源的理解类比为数据库中的一行数据

-   控制器中 **不应该** 出现私有的方法, 控制器中的代码 **应该** 越短越好

-   控制器中方法的命名规则 **应该** 和服务层的方法命名保持一致

-   控制器方法 **应该** 响应`Response`对象

-   验证类型的异常抛出 **应该** 在控制器层抛出

##### Service

服务层是对控制器层的扩展, 它作为`M(Model)` 和 `C(Controller)`的中间桥梁, 是业务逻辑的主要实现目录, 一般情况下, 服务层的目录结构 **应该** 保持和控制器层一致

示例如下:

```php
├── app
   ├── Controller # 控制器目录
   │   ├── Material # 素材类控制器
   │   │   ├── ItemController # 商品控制器
   │   │   ├── IconController # 图标控制器
   │   ├── Shop
   └── Service # 服务层目录
       ├── Material # 素材类服务
       │   ├── ItemService # 商品服务类
       │   ├── IconService # 图标服务类
       ├── Shop
```

-   服务层的方法 **绝不** 能返回`Response`对象
-   服务层的入参 **不应该** 传入数组, **应该** 传入具体对象 或 具体实参
-   **绝不** 在服务层调用 或 注入控制器对象, 除了降低耦合外, 还有一种场景下, 这种骚操作大概率会导致 依赖注入的死循环



##### Model

数据层与逻辑层中间, **可以** 增加一层**数据服务层**, 数据层(Model层)中 **应该** 只存在不涉及业务的CURD操作及其他通用方法的封装, **数据服务层**用来处理 `S(Service)` 和 `M(Model)` 间的调用, 数据服务层是包含业务逻辑处理的

-   模型层 **应该** 列出所有表中的字段、关联的类型
-   模型层 **应该** 继承自 `huanhyperf/database` 的模型基类, 而不是继承自框架的模型基类, 这会导致某些特殊关联失效
-   表的枚举常量 **应该** 在模型层维护在模型内部
-   表中的json或数组字符串 **应该** 使用 Cast 转换, 而 **不应该** 手动转换
-   数据表的CURD **应该** 使用模型层的 Builder  而 **不应该** 使用 Query, Query更新不会触发数据库事件, 且更新的数据条数是不可控的
-   数据模型的关联 **应该** 有默认值

### 依赖注入

-   **不应该** 使用 `@Value` 来注入 `apollo`可以能会改动的配置信息
-   依赖注入 主要有构造函数注入 和注解注入, **应该** 统一使用注解注入

### 注解的使用

### 日志的输出

日志的输入 **必须** 严格按照等级输出

```php
# debug级别 应该只在本地开发使用
const DEBUG = 100;
# 普通信息级别 用户登录 SQL语句等
const INFO = 200;
# 通知信息级别 这个信息值得关注
const NOTICE = 250;
# 警告信息级别 发生了可预见的错误, 业务流没有按照预期的链路走
const WARNING = 300;
# 运行错误级别 这个错误可能会影响主流程
const ERROR = 400;
# 关键的错误 这个错误必须马上解决
const CRITICAL = 500;
# 紧急的错误 这个错误必须立刻、马上解决
const ALERT = 550;
# 致命的错误 这个错误必须立刻、马上、赶紧解决
const EMERGENCY = 600;
```

-   日志的记录长度 **不应该** 过长, 且频率 **不应该** 太高
-   DEBUG级别的日子 **应该** 只在本地使用
-   容器内日志显示最多500行, ELK最多传输长度1500Byte, 所以 **不应该** 在dev及以上环境中输出过长、频次过高的日志, 它会增加debug的难度

### 命令行脚本

-   命令行脚本命名 **必须** 依照 `命名空间:操作命令` , `操作命令` 多个单词的命名时 **必须** 以英文的`-`连接

### 路由

路由的入口文件为`config/routers.php` 除了基础的路由, 其他的路由 **必须** 维护在 `config/routers` 目录下

路由的命名除去`/`外, **必须** 只用小写字母和 单词连接 **必须** 使用英文状态下的中划线 `-` 来命名

#### 路由的划分

-   路由命名 应该 遵循多级命名空间方法, 如 控制器 `app/Controller/Desc/DescConfigController.php`  对应 路由: `/api/desc/desc-config` `api` 区分业务类型, `desc` 区分控制器二级目录 `desc-config` 区分所在的控制器, 再加以资源路由划分方法

    >   这样做的好处是在debug的时候, 看到路由就可以快速定位到所在控制器, 对于业务划分 和 熟悉业务也有一定的帮助, 杂乱无章的命名方法 应该 被扼杀在摇篮里

-   路由命名 **必须** 要明显区分业务类型或模块, 如接口类型 **应该** 以 `api` 为命名空间, 后台接口 **应该** 以 `admin` 为命名空间等, 默认的业务 **可以** 省略一级命名空间

-   路由在 `config/routers` 目录下 必须 按主业务类型或模块划分后 按照文件存储



### 表单验证

前端的请求数据原则上 **应该** 都要做一层数据校验, 数据校验的规则及写法 参考`validator`组件, hyperf的表单验证借鉴于laravel, 文档可以参考laravel更详尽的文档

>   [Laravel 可用的表单验证规则](https://learnku.com/docs/laravel/8.0/validation/1302#available-validation-rules)

### 中间件开发

### 数据迁移

数据库的结构变动 **应该** 都由迁移创建以便于保持版本控制, 当迁移的文件过多时, **应该** 将存量迁移转换为sql备份, 根据对表结构的操作, 操作类型 **应该** 区分为以下几种类型:

-   增加表 create table
-   删除表 drop table
-   表字段的操作:
    -   增加 add column
    -   修改 modify column
    -   删除 drop column
-   索引的操作:
    -   增加索引 add index
    -   修改索引 modify index
    -   删除索引 drop index

#### 数据库迁移的命名规范

命名 **应该** 根据操作类型来命名:

-   新建表 create_table_{table_name}

-   修改表

    -   修改字段 alter_{table_name} _modify_column

-   增加字段 alter_{table_name} _add_column

    -   删除字段 alter_{table_name} _drop_column

-   增加索引 alter_{table_name} _add_index

    -   删除索引 alter_{table_name} _drop_index

-   修改索引 alter_{table_name} _modify_index

    -   混合操作 alter_{table_name} _mixed

-   删除表 drop_table_{table_name}

    ​



#### 存量数据的备份

存量数据的备份文档待完善

###