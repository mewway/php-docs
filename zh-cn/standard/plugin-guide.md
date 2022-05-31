# 基础扩展

扩展是可以复用于多个场景的多个函数或方法的集合, 它是服务于业务场景的特殊实现, 如 文件操作、 图片操作、 文档导入导出、 http响应约定类、 分页器等等, 都 **可以** 封装为扩展, 在一个项目内多次复用

区分应该封装为Composer组件或扩展的依据 **可以** 参考如下:

1.   组件是对业务特定场景 或 功能集合的特定实现, 它适合较为复杂的实现, 需要依赖其他实现的组件, 可以在多个项目中使用, 这种情况下 **应该** 封装为Composer组件
2.   扩展相对于组件则较为轻量级, 它是功能较为简单 或者 只能在一个项目里多次复用的代码集合, 这种情况下 **应该** 封装为扩展
3.   扩展和组件没有明确的边界和限定, 可以视项目而定

## 扩展开发的准则

- 抽象处理, 尽可能适用于各种场景

## 组件的开发
一个composer组件 **应该** 包含以下目录
```php
├── src # 组件的源码目录
├── bin # 脚本执行目录
├── config # 配置目录
├── tests # 单元测试目录
└── migrations # 需要迁移或发布的配置文件
```
在 `hyperf` 框架下开发组件大同小异， hyperf的组件src下一般都包含`ConfigProvider` 类 其结构如[Config Provider](#Config Provider)

> [Hyperf框架组件开发指南](https://hyperf.wiki/2.2/#/zh-cn/component-guide/intro)

## Config Provider

```php
<?php

namespace Hyperf\Foo;

class ConfigProvider
{
    public function __invoke(): array
    {
        return [
            // 合并到  config/autoload/dependencies.php 文件
            'dependencies' => [],
            // 合并到  config/autoload/annotations.php 文件
            'annotations' => [
                'scan' => [
                    'paths' => [
                        __DIR__,
                    ],
                ],
            ],
            // 默认 Command 的定义，合并到 Hyperf\Contract\ConfigInterface 内，换个方式理解也就是与 config/autoload/commands.php 对应
            'commands' => [],
            // 与 commands 类似
            'listeners' => [],
            // 组件默认配置文件，即执行命令后会把 source 的对应的文件复制为 destination 对应的的文件
            'publish' => [
                [
                    'id' => 'config',
                    'description' => 'description of this config file.', // 描述
                    // 建议默认配置放在 publish 文件夹中，文件命名和组件名称相同
                    'source' => __DIR__ . '/../publish/file.php',  // 对应的配置文件路径
                    'destination' => BASE_PATH . '/config/autoload/file.php', // 复制为这个路径下的该文件
                ],
            ],
            // 亦可继续定义其它配置，最终都会合并到与 ConfigInterface 对应的配置储存器中
        ];
    }
}
```
