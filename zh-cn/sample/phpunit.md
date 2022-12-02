# 单元测试的最佳实践指南
> 最佳实践指南基于 hyperf框架 及 PHPStorm IDE的基础上


## IDE中的配置

### PHPStorm

> 正确配置单元测试框架

![单元测试配置](../../resource/phpunit-config.png)

> 测试模版配置

![单元测试模版配置](../../resource/phpunit-config2.png)

### Bootstrap File

> 正确配置初始化脚本
```php
declare(strict_types=1);
/**
 * This file is part of Hyperf.
 *
 * @link     https://www.hyperf.io
 * @document https://doc.hyperf.io
 * @contact  group@hyperf.io
 * @license  https://github.com/hyperf/hyperf/blob/master/LICENSE
 */
ini_set('display_errors', 'on');
ini_set('display_startup_errors', 'on');

error_reporting(E_ALL);
date_default_timezone_set('Asia/Shanghai');

! defined('BASE_PATH') && define('BASE_PATH', dirname(__DIR__, 1));
$hook = version_compare(swoole_version(), '4.6.0') >= 0 ? SWOOLE_HOOK_ALL ^ SWOOLE_HOOK_SOCKETS : SWOOLE_HOOK_ALL;
!defined('SWOOLE_HOOK_FLAGS') && define('SWOOLE_HOOK_FLAGS', $hook);

require BASE_PATH . '/vendor/autoload.php';

Hyperf\Di\ClassLoader::init();

$container = require BASE_PATH . '/config/container.php';

$container->get(Hyperf\Contract\ApplicationInterface::class);
```
为了保持目录的一致，生成单元测试文件都 **应该** 通用IDE默认来生成
> `option` + `1` 快速定位文件位置， 右键 `New` 来生成测试文件，保持目录规则一致

![生成单元测试文件](../../resource/phpunit-config3.jpg)

## 单元测试场景
根据目前框架中单元测试划分几个典型的使用场景
### 接口测试
一般的接口开发中，一般的测试方法通过Postman类似的工具、或者curl命令行进行测试，这种场景下必须启动框架，并且后来的小伙伴无法对既往测试
的数据有所感知，所以建议在单元测试中，通过继承 `HttpTestCase` 基类来实现接口测试，接口测试中，常见的场景及注意点如下：
- 对数据校验规则的校验
- 对期望的判断
- 有先后顺序、先后依赖的参数构建

> 批量生成数据可以通过数据供给器 `dataProvider` 来实现 有先后顺序依赖的 可以通过 `depends` 来实现
 
在对数据规则做校验测试的时候 数据供给器 就能起到快速、便捷的作用
例如可以定义如下数据供给器：
```php
public function formDataProvider()
{
    return [
        'case1' => ['param1' => 1],
        'case2' => ['param1' => 2],
    ];
}
```
并在后续的测试方法中， 将数据来源传入需要的方法中, 在该方法中，将会根据数据供给器的参数数量， 多次调用该测试方法
```php
/**
 * @dataProvider formDataProvider
 */
public testMethod1()
{
}
```
### 
