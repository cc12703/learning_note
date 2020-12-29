

# Git开发方式

## 分支管理

### master（主分支）
* 代码处理可发布状态
* 所有tag基于该分支生成


### develop（开发分支）
* 代码处理最新开发状态
* 所有开发者都在该分支进行开发

### feature（特性分支）
* 用于开发一个独立功能
* 起源于develop, 合并到deveop
* 在开发者本地，无需上传中心库

### release（发布分支）
* 用于修复测试中出现的bug
* 起源于develop，合并到master
* bug修改同步到develop
* 在新功能开发完成，调试、自测结束后，进入正式测试前创建
* 在测试结束后，合并完成后，进行删除

### hotfix
* 用于修改关键bug
* 起源于master, 合并到master, develop 
* 在开发者本地，无需上传中心库


## log规范
### 概述
* 不是给最终用户看的，而是给开发维护人员看的
* 需要提供涉及项目的一切必要信息：过程的、质量的、工具的
* 配置模板：git config –global commit.template ~/log.tmp
* 每次提交限定于完成一次逻辑功能，每次更新都尽可能小，更易于理解

### 模板
> [Category] Title content
>                     -- 第二行要为空行
>     Body content
> [BUGID] QC#733 | JIRA#766     —— bug ID
> [REVIEWID] RB#767             —— 代码评审ID

#### Category
* BUGFIX     修改bug
* FEATURE    添加新功能
* REFACT     重构代码，不修改功能
* URGENT     紧急提交
* TASK       前提操作：修改版本号
* IMPROVE    改进现有功能