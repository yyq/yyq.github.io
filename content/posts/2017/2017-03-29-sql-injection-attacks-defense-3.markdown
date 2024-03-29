---
Title: 《SQL注入攻击与防御》之防御
author: Young
date: 2017-03-29
tags: [security, sql, injection, attack, defense]
Status: public
---

### 代码层防御

#### 领域驱动的安全性

* SQL注入之所以发生，是因为我们的应用程序不正确的将数据在不同表示方式之间进行映射
* 通过将数据封装到有效的值对象中，并限制对原始数据的访问，我们就可以控制对数据的使用

#### 使用过参数化语句

* 动态SQL（或者将SQL查询组装成包含受用户控制的输入的字符串并提交给数据库）是引发SQL注入漏洞的主要原因
* 应该使用参数化语句（也叫做预处理语句）而非动态SQL来安全的组装SQL查询
* 在提供数据时可以只使用参数化语句，但却无法使用参数化语句来提供SQL关键字或标识符（比如表名或者列名）

#### 验证输入

* 尽可能坚持使用白名单输入验证（只接收期望的已知良好的输入）
* 确保验证应用受到的所有受用户控制的输入的类型，大小，范围和内容
* 只有当无法使用白名单输入验证时才能使用黑名单输入验证（拒绝已知不良的或基于签名的输入）
* 绝不能单独只使用黑名单检验数据。至少应该总是将它与输出编码技术一起结合使用

#### 编码输出

* 确保对包含用户可控制输入的查询进行正确编码以防止使用单引号或其他字符来修改查询
* 如果正在使用LIKE子句，请确保对LIKE中的通配符恰当的编码
* 在使用从数据库接收到的数据之前确保已经对数据中的敏感内容进行了恰当的输入验证和输出编码

#### 规范化

* 将输入编码或变味规范格式后才能执行输入验证过滤器和输出编码
* 请注意，任何单个字符都存在多种表示方式以及编码方法
* 尽可能使用白名单输入验证并拒绝非规范格式的输入

#### 通过设计来避免SQL注入的危险

* 使用存储过程以便在数据库层拥有较细粒度的许可
* 可以使用数据访问抽象层来对整个应用施加安全的数据访问
* 设计时，请考虑对敏感信息进行附加的控制

### 平台层防御

#### 使用运行时保护

* 无法修改代码时，运行时保护是应对SQL注入的一种有效技术
* 如果调整得当，Web应用防火墙可以有效监测，缓和和预防SQL注入
* 运行时保护可以跨越多层，多级，其中包括网络，web服务器，应用程序框架以及数据库服务器

#### 确保数据库安全

* 加固数据库虽然无法完全阻止SQL注入，但却可以显著降低其影响
* 应该只将攻击者沙箱化在应用程序所用数据上。在锁定的数据库服务器中，不应该影响所连接网络上的其他数据库和系统
* 应该将访问局限在必须的数据库对象上，比如存储过程只授予EXECUTE许可，此外，对敏感数据明智的使用强加密技术可以防止未经验证的数据访问

#### 额外的部署考虑

* 加固过的web层部署和网络架构无法完全阻止SLQ注入，但却可以显著降低其影响。
* 面对自动攻击者的威胁（比如SQL注入蠕虫），尽量减少网络，web和应用程序级别上的泄露，将有助于减少被发现的机会
* 架构得当的网络应该只允许使用验证过的连接来连接数据库服务器，并且数据库服务器自身不应该产生带外连接

### 确认并从SQL注入攻击中恢复

#### 调查可疑的SQL注入攻击

* 只能由计算机安全事件响应人员和组织中已授权执行调查的取证专家，来执行SQL注入攻击的调查取证工作。

#### 合理的调差取证实践要求

* 在调查取证期间，应该对收集到的所有文件做到真正bit-for-bit复制
* 应该为复制的每一个文件生成哈希值，并与源文件的哈希值进行比较，以验证bit-for-bit复制的完整性
* 将调查取证期间执行的所有操作记录在文档中，包括对RDBMS执行的所有查询和返回的查询结果
* 确保所有收集到的文件写入洁净的存储介质中，并保存在一个安全的地方

#### 分析数字化痕迹

* 数字化痕迹就是相关数据的集合
* 对于SQL注入攻击的调查取证，下列痕迹非常有用：web服务器的日志文件，数据库执行计划，事务日志和数据库对象的时间戳

#### 识别SQL注入攻击活动 

* 对web服务器的日志文件执行一次广泛的分析，查找异常偏高的web请求数量出现的日期，或者web服务器与客户端计算机之前带宽利用率异常偏高的日期
* 检查数据库执行计划和相关日志，查找恶意查询
* 检查食物日志，寻找在攻击时间段内出现的可疑活动，重点关注已执行的INSERT,UPDATE和DELETE语句
* 检查数据库对象的时间戳，以识别用户账号的创建，权限提升和表的创建操作

#### 确认SQL注入攻击是否成功

* 如果发现下列情况，就可以确认一次SQL注入攻击已经成功
 * 在数据库的执行计划或相关数据库日志中捕获了SQL注入的活动
 * 在未经授权的情况下，创建或修改了事物或对象

#### 遏制安全事件

* 拔除受损害数据库和相应web服务器的网线

#### 评估涉及的数据

* 必须对数据进行评估，以确保组织机构确定适当的监管和法律要求

#### 通知正确的人员

* 应该由受损害组织机构的高级管理人员和法律顾问对损害通报进行管控

#### 确定攻击者在系统上执行的操作

* 可以通过对数据库进行取证，确定攻击者在攻击期间执行的具体操作

#### 确定攻击的有效载荷

* 备份受害的数据库
* 提取恶意的SQL注入查询
* 检查并理解恶意查询的逻辑，从而理解攻击载荷企图实现什么目的
* 搜索对恶意查询的引用
* 确定已识别的恶意查询是否属于静态或动态攻击载荷的一部分
* 查找多种漏洞
* 应该根据承载的是静态载荷还是动态载荷对攻击进行分类
* 根据攻击载荷确定如何从安全事件中恢复

#### 从SQL注入攻击中恢复

* 将数据库恢复到已知的良好状态
* 检验数据库服务器的配置
* 识别并修复SQL注入漏洞
* 在线恢复系统并恢复web服务