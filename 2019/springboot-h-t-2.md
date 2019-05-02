# Springboot + Spring security + hibernate开发 (2)
*Spring security简单上手， 本文作为自己的笔记*

### 1 - Spring Security介绍
&emsp;&emsp;Spring Security提供了一个简单方便的权限框架，主要按角色，身份实现拦截控制等操作。但是具体定制方面有限制，适合比较简单的应用。

Principle(User), Authority(Role)和Permission是Spring Security的3个核心概念。
跟通常理解上Role和Permission之间一对多的关系不同，在Spring Security中，Authority和Permission是两个完全独立的概念，两者并没有必然的联系，但可以通过配置进行关联。


应用级别的安全主要分为“验证( authentication) ”和“(授权) authorization ”两个部分。

这也是Spring Security主要需要处理的两个部分：
在Spring Security中，认证过程称之为Authentication(验证)，指的是建立系统使用者信息( principal )的过程。使用者可以是一个用户、设备、或者其他可以在我们的应用中执行某种操作的其他系统。
" Authorization "指的是判断某个 principal 在我们的应用是否允许执行某个操作。在 进行授权判断之前，要求其所要使用到的规则必须在验证过程中已经建立好了。
这些概念是通用的，并不是只针对"Spring Security"。


### 2 - 角色：权限对照表

### 3 - UserDetail，WebSecurityConfig配置
### 4 - 实现登录页面