# Springboot + Spring security + hibernate开发 (2)
*Spring security简单上手， 本文作为自己的笔记*

### 1 - Spring Security介绍
&emsp;&emsp;Spring Security提供了一个简单方便的权限框架，主要按角色，身份实现拦截控制等操作。但是具体定制方面有限制，适合比较简单的应用。

Principle(User), Authority(Role)和Permission是Spring Security的3个核心概念。
跟通常理解上Role和Permission之间一对多的关系不同，在Spring Security中，Authority和Permission是两个完全独立的概念，两者并没有必然的联系，但可以通过配置进行关联。

ROLE即角色，比如管理员和员工，但是Spring security中并没有多个操作权限对应角色这样的关系，需要注意。不过我们可以自己设置这种一对多的关系，后文举例介绍怎么设置角色与权限。

应用级别的安全主要分为“验证( authentication) ”和“(授权) authorization ”两个部分。

authentication 即验证使用者的身份`principal`,authorization则是根据身份查询到其具有的ROLE，可以是多个ROLE，以进行拦截操作。

**简单来说我们需要一个角色表，一个WebConfig确定权限逻辑，一个UserDetail验证登录**

### 2 - 角色：权限对照表
---

User表：
|用户名|密码|角色|
|---|---|---|
|user|password|role

Role表：
|角色|权限描述|
|---|---|
|role|description|

如果不需要一对多，上面的就足够了，如果需要一对多，那么要加一个权限表，以及索引表。
User表：
|index|角色|权限|
|---|---|---|
|id|role|permission|

*根据索引查找角色具有的权限*
Permission表：
|权限|权限描述|
|---|---|
|permission|description|


### 3 - Spring Security 项目结构
---
对于Servlet Application：

### 4 - UserDetail，WebSecurityConfig配置
---


### 5 - 实现登录页面
---

