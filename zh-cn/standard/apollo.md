# Apollo配置规范参考

阿波罗配置中， 不同项目 **应该** 区分前台和后台的容器，并配置不同的命名空间以供不同场景下使用

命名空间配置规范 **可以** 参考如下：
1. application 前后台容器重写配置
2. ci-params jenkins构建参数
3. {project-name}-startup-config 前台容器配置
4. {project-name}-worker-startup-config 后台容器配置
5. view 前端页面配置

> Q：startup-config 和 application 的优先级配置

```text
  A： 配置应该在容器配置中优先覆盖， application只用
   于少部分场景下的配置修改， 即application 不能
   包含过多的配置信息
```
