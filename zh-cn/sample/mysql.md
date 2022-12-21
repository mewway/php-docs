# 数据库-最佳实践指南

## 数据库设计

数据库设计 **必须** 使用框架自带的 `migration` 来实现，**绝不** 使用其他第三方数据库软件进行结构变更
> 为了数据库版本可控可追溯， 结构的修改和新增来源建议仅有一个， 避免多个来源导致不同环境间的数据结构不一致

一个简单的migration例子：
```php
class CreateUserStore extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('user_store', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->bigInteger('org_id')->default(0)->comment('租户id');
            $table->bigInteger('user_id')->default(0)->comment('用户id');
            $table->bigInteger('store_id')->default(0)->comment('店铺id');
            $table->bigInteger('auth_by')->default(0)->comment('授权者');
            $table->timestamps();
            $table->index(['org_id', 'user_id'], 'org_id_user_id');
            $table->comment('用户店铺授权关联表');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('user_store');
    }
}
```

一个字段修改的例子：
```php
class ModifyGoodsImportDetailExtraFieldChangeType extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        \App\Model\ShardingModel::shouldCallShardingMigration('goods_import_detail', 100, function($table) {
            Schema::table($table, function(Blueprint $table) {
                $table->mediumText('extra')->comment('扩展字段')->default('')->change();
            });
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
    }
}
```

## 注释的规范
注释的规范可以避免很多的麻烦， 如枚举值的含义及表的名称，字段的意义，对于理解业务是有很大的帮助的
> 现存的表设计中， 表注释的确实 与 表名命名的类似， 导致业务理解晦涩困难

一个简单明了的表注释应该如下例子：
```php
$table->tinyInteger('status')->default(0)->comment('任务状态 0-未开始 1-解析中 2-成功 3-失败');
$table->tinyInteger('enable')->default(1)->comment('是否可用 0-不可用 1-可用');
```

注释的规范参考及建议见[数据库规范](zh-cn/standard/mysql.md)