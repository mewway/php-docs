# 框架使用-最佳实践指南

## 控制器
对应的目录为 `app/Controller`
控制器中，最为关键的要素即为参数的校验，`hyperf/validation`组件继承自 `laravel`，它可以轻松完成任意维度、任意类型的数据校验，并且支持的规则强大、简单易用
> 参数校验的参考文档[validation](https://learnku.com/docs/laravel/6.x/validation/5144#c58a91)

简单的参考例子如下：
```php
    public function brandOptions(RequestInterface $request)
    {
        $query = $request->all();
        $rule = [
            'store_id' => 'required|string|numeric',
            'keyword' => 'nullable|string',
            'category_id' => 'nullable|string',
            'force_update' => 'nullable|in:0,1',
        ];

        $message = [];
        $validator = $this->validator->make($query, $rule, $message);
        if ($validator->failed()) {
            throw new ApiException(AppCode::VALIDATION_FAILED, $validator->errors()->first());
        }

        $storeIds = Common::splitStr((string)$query['store_id']);
        $forceUpdate = $query['force_update'] ?? 0;
        $categoryIds = empty($query['category_id']) ? [] : Common::splitStr((string)$query['category_id']);
        $data = $this->queryAdapter->brandOptions($storeIds, $query['keyword'] ?? '', $categoryIds, $forceUpdate);

        return $this->response->success($data);
    }
```
## 异常处理
对应的目录为 `app/Exception`
异常设计 **应该** 要区分类型，数据验证异常 和 接口异常区分开来， 异常抛出要按业务区分
例如数据验证错误只在控制器层中抛出，接口异常只在逻辑层抛出
## 队列
对应的目录为 `app/Job`
异步队列设计 **应该** 要支持自分发， 无需每次获取`DriverInterface`， 再做入队
一个自分发队列参考例子如下：
```php
namespace App\Job;

use App\Component\ContextThrough;
use App\Constants\AppCode;
use App\Exception\BusinessException;
use Hyperf\AsyncQueue\Driver\DriverFactory;
use Hyperf\AsyncQueue\Driver\DriverInterface;
use Hyperf\AsyncQueue\Job;

abstract class AbstractJob extends Job
{
    /**
     * @var null|string
     */
    protected $bindQueue;

    /**
     * @var null|int
     */
    protected $delay;

    protected $context = [];

    /**
     * 执行需要入队的队列名称
     */
    public function queue(string $queue): self
    {
        $this->bindQueue = $queue;

        return $this;
    }

    /**
     * 处理的延时
     */
    public function delay(int $delay): self
    {
        $this->delay = $delay;

        return $this;
    }

    /**
     * 最大重试次数
     */
    public function maxAttempts(int $max): self
    {
        $this->maxAttempts = $max;

        return $this;
    }

    /**
     * 自分发方法
     */
    public function dispatch(?int $delay = null, ?string $queue = null, ?int $max = null): bool
    {
        $this->context = ContextThrough::getThrough();
        ! is_null($max) && $this->maxAttempts = $max;
        $driver = $this->getQueueDriver($queue ?? $this->bindQueue);

        return $driver->push($this, $delay ?? $this->delay ?? 0);
    }

    /**
     * 支持上下文穿透的处理方法
     */
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

    /**
     * 队列处理的入口方法
     *
     * @return mixed
     */
    abstract public function execute();

    /**
     * 获取队列的配置
     */
    private function getQueueConfig(?string &$queueName = null): array
    {
        $queueName ??= 'default';
        $config = config('async_queue.' . $queueName, []);
        if (empty($config) || ! isset($config['channel'])) {
            throw new BusinessException(AppCode::SYS_DEFAULT_ERROR, sprintf('queue %s or its channel not exists', $queueName));
        }

        return $config;
    }

    /**
     * 获取入队的适配器
     */
    private function getQueueDriver(?string $queueName = null): DriverInterface
    {
        $this->getQueueConfig($queueName);

        return make(DriverFactory::class)->get($queueName);
    }
}
```
## 事件
对应的目录为 `app/Event` 事件， `app/Listener` 监听
事件设计 **应该** 要支持自分发，而不是每次都需要实例化 `EventDispatcherInterface` 再去分发事件

一个自分发示例如下：
```php
/**
 * @method  static dispatch():self
 */
abstract class AbstractEvent
{
    public function __call(string $name, array $params)
    {
        if ('dispatch' === $name) {
            return $this->getEventDispatcher()->dispatch($this);
        }
    }

    public static function __callStatic(string $name, array $params)
    {
        if ('dispatch' === $name) {
            $self = new static(...$params);
            $self->dispatch();
        }
    }

    private function getEventDispatcher()
    {
        return make(EventDispatcherInterface::class);
    }
}
```
## 模型
对应的目录为 `app/Model`
模型中需要注意的几个要素：
- 注释中对关联和表字段的注释要描述清楚且类型正确
- 模型的关联需要说明清楚
- 简单的类型转换 **应该** 使用`cast` 而不是 获取器 或 修改器
- 表述枚举的常量应该定义在模型中直接定义
- 常用方法与业务无关的应该直接定义在模型内 或 在父类基础模型中定义

示例如下： 
```php
/**
 * @property int $id
 * @property int $platform_id
 * @property array $extra
 * @property SupportedPlatform $platformInfo
 * @property HasMany|SubAccount[]|Collection $subAccount
 * @property Organization|null $orgInfo
 */
class Store extends Model
{
    const STATUS_ENABLE = 1;//启用状态
    /**
     * @var string
     */
    protected $table = 'store';
    /**
     * @var array
     */
    protected $fillable = [
        'platform_id',
        'platform_user_id',
        'extra',
    ];
    /**
     * @var array
     */
    protected $casts = [
        'id' => 'integer',
        'platform_id' => 'integer',
        'extra' => 'array',
    ];

    /**
     * @return \Hyperf\Database\Model\Relations\BelongsTo
     */
    public function platformInfo()
    {
        return $this->belongsTo(SupportedPlatform::class, 'platform_id', 'platform_id');
    }

    /**
     * @return \Hyperf\Database\Model\Relations\BelongsTo
     */
    public function orgInfo()
    {
        return $this->belongsTo(Organization::class, 'org_id', 'org_id');
    }
```
## 中间件
对应的目录为 `app/Middleware`
中间件可以用来做参数过滤、跨域处理、登录鉴权等其他权限控制相关、处理输入输出等等
## 常量
对应的目录为 `app/Constants`, **绝不** 在此目录写入任何业务代码， 常量区分不宜过多
