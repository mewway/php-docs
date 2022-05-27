# 编码风格

## 代码风格

编码规范 **必须** 严格参照`PSR-1`、`PSR-12`的规范要求


### PSR规范

`PSR` 是由 PHP FIG 组织制定的 PHP 规范，是 PHP 开发的实践标准, `PSR`规范是我们主要遵守的编码规范, 阅读时主要参照:

-   PSR-1 基础编码规范
-   PSR-4 自动加载的规范
-   PSR-12(替换扩展PSR-2) 编码规范的扩充

>   [PSR规范参考](https://learnku.com/docs/psr)

## PHP-CS-FIXER

php-cs-fixer是项目中使用的代码格式化工具, 可以配合IDE使用, 项目中 **应该** 使用本工具对代码进行格式化, 项目中使用的`php-cs-fixer`的规则及配置参考如下, 项目中新增 `.php-cs-fixer.{env}.php`, 在配置文件中写入如下, 并 **应该** 纳入版本管理

```php
declare(strict_types=1);

$header = <<<EOF
This file is part of HuanLeGuang Project, Created by php-cs-fixer 3.0.
EOF;

$finder = PhpCsFixer\Finder::create()
    ->exclude('tests/Fixtures')
    ->exclude('public')
    ->exclude('runtime')
    ->exclude('vendor')
    ->in(__DIR__)
    ->append([
        __DIR__.'/dev-tools/doc.php',
        // __DIR__.'/php-cs-fixer', disabled, as we want to be able to run bootstrap file even on lower PHP version, to show nice message
        __FILE__,
    ]);

$config = new PhpCsFixer\Config();
$config
    ->setRiskyAllowed(true)
    ->setRules([
        '@PHP71Migration:risky' => true,
        '@PHPUnit75Migration:risky' => true,
        '@PhpCsFixer' => true,
        '@PhpCsFixer:risky' => true,
        '@PHPUnit75Migration:risky' => true,
        'no_whitespace_in_blank_line' => false,
        'no_blank_lines_after_class_opening' => false,
        'no_trailing_whitespace_in_comment' => false,
        'phpdoc_add_missing_param_annotation' => ['only_untyped' => false],
        'phpdoc_summary' => false,
        'general_phpdoc_annotation_remove' => ['annotations' => ['expectedDeprecation']], // one should use PHPUnit built-in method instead
        'ordered_imports' => ['sort_algorithm' => 'alpha', 'imports_order' => ['const', 'class', 'function']],
        'not_operator_with_successor_space' => true,
        'concat_space' => ['spacing' => 'one'],
        'multiline_whitespace_before_semicolons' => ['strategy' => 'no_multi_line'],
        'header_comment' => ['header' => $header],
    ])
    ->setFinder($finder);

return $config;
```

php-cs-fixer的使用指南参照github 文档

>   [PHP-CS-FIXER 规则列表](https://github.com/FriendsOfPHP/PHP-CS-Fixer/blob/master/doc/ruleSets/index.rst)
