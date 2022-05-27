# 项目文档编写规范

项目中 **必须** 包含 `readme.md`, 书写规范参照 `Markdown` 语法

-   readme 中 **应该** 包含项目的简介、 背景及相关的文档链接
-   readme 中还 **应该** 包含一些关键的命令行使用方法

项目中 **应该** 包含`changelog.md`, 规范同 `readme.md`, `changelog` 的存在可以大幅提升新接受的研发小伙伴对项目的了解

-   标准的changelog **可以** 包含以下内容:

    1.   Version 版本号和版本的基础信息
    2.   Backward Incompatible Change 向下不兼容的改动
    3.   Feature 新功能改进, **可以** 用 `Experimental` 加以区分试验性的功能
    4.   Improvement 功能提升优化, 性能或功能优化
    5.   Bug Fixes 缺陷的修复

`changelog` 模版参照:

```markdown
## 欢乐逛 V1.2.3 2021-11-20
版本发布的描述及大致功能的描述
### Feature
	- 新发布了功能点1
	- 新发布了功能点2
### Improvement
	- 提升了上传图片的性能
### Bug Fixes
	- 修复了bug1
### Backward Incompatible Change
	- 图片上传不再支持单图

```

