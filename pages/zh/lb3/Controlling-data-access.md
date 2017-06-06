---
title: "控制数据的访问"
lang: en
layout: navgroup
navgroup: access-control
keywords: LoopBack
tags: authentication
sidebar: zh_lb3_sidebar
permalink: /doc/zh/lb3/Controlling-data-access.html
summary: LoopBack 使用 ACL 来控制数据的访问权限
---

## 启用登录认证

当你使用 [LoopBack 命令行](Application-generator.html) 创建一个应用后，访问控制是默认开启的，_除非_ 你选择创建的是 "empty-server" 空服务应用类型。
如果你需要给 "empty-server" 应用添加访问控制，你需要添加一个 boot
script 启动脚本调用 `enableAuth()`。例如，创建一个 `server/boot/authentication.js` 文件，并包含以下内容：

```javascript
module.exports = function enableAuthentication(server) {
  server.enableAuth();
};
```

## 指定用户的角色

在给用户指定角色之前，你需要先确定你的应用都需要哪些角色。
大多数应用都会有两个相对的概念：un-authenticated 或 anonymous users (未登录用户) 和 authenticated users (已登录用户)。
另外，大多数应用都有一个管理员角色拥有最多的访问权限，同时可以设置一定数量的其他非管理员角色。

比如，这个示例程序 [startkicker](https://github.com/strongloop/loopback-example-access-control)
拥有四种类型的用户: `guest`, `owner`, `team member` 和 `administrator`。
每种用户类型基于其角色和 ACL，可以访问应用内特定模块。

### 用户访问类型

LoopBack 提供了一个内建的用户模型 [User](https://apidocs.strongloop.com/loopback/#user) 及其对应的 [REST API](User-REST-API.html) 。这个用户模型继承了 [PersistedModel object](https://apidocs.strongloop.com/loopback/#persistedmodel)
所有的增删改查 "CRUD" (create, read, update, and delete) 方法。
该用户模型的每个 CRUD 方法都对应到 `READ` 或 `WRITE` 两种访问类型 (Access Type) 中的一种:

对应到 `READ` 访问类型的方法如下:

* [exists](https://apidocs.strongloop.com/loopback/#persistedmodel-exists) - 一个布尔方法用来确认一个用户是否存在。
* [findById](https://apidocs.strongloop.com/loopback/#persistedmodel-findbyid) - 用 ID 查找一个用户。
* [find](https://apidocs.strongloop.com/loopback/#persistedmodel-find) - 查找所有符合条件的用户。
* [findOne](https://apidocs.strongloop.com/loopback/#persistedmodel-findone)  - 查找一个符合条件的用户。
* [count](https://apidocs.strongloop.com/loopback/#persistedmodel-count) - 返回所有符合条件的用户总数。

对应到 `WRITE` 访问类型的方法如下:

* [create](http://apidocs.strongloop.com/loopback/#persistedmodel-create) - 创建一个用户。
* [updateAttributes](http://apidocs.strongloop.com/loopback/#persistedmodel-updateattributes) (update) - 更新一条用户记录。
* [upsert](http://apidocs.strongloop.com/loopback/#persistedmodel-upsert) (update or insert) - 更新或创建一条用户记录。
* [destroyById](https://apidocs.strongloop.com/loopback/#persistedmodel-destroybyid) (equivalent to removeById or deleteById) - 删除指定 ID 的用户。

该用户模型剩下的其他方法，默认的访问类型均为 `EXECUTE`；比如, 一个自定义方法的默认访问类型为 `EXECUTE`。

## 定义访问控制

使用 [ACL generator](ACL-generator.html) 设置访问控制规则

示例应用 [loopback-example-access-control](https://github.com/strongloop/loopback-example-access-control) 展示了如何清晰定义用户及其权限：

* Guest - 访客
  * Role = $everyone, $unauthenticated
  * 只能访问 "List projects" 功能
* John - 项目拥有者
  * Role = $everyone, $authenticated, teamMember, $owner
  * 可以访问所有功能，除了 "View all projects"
* Jane - 项目团队成员
  * Role = $everyone, $authenticated, teamMember
  * 可以访问所有功能，除了 "View all projects" and "Withdraw"
* Bob - 管理员
  * Role = $everyone, $authenticated, admin
  * 可以访问所有功能，除了 "Withdraw"

一旦你确定了类似的清单，你可以轻松的使用命令行为一个应用设置访问控制规则，具体方法如下：

## 使用 ACL Generator 设置访问控制规则

使用 [ACL generator](ACL-generator.html) 可以为一个应用设置静态的访问控制规则，命令行如下：

```shell
$ lb acl
```

## 应用访问控制规则

每个访问请求会被映射到一个 JSON 对象，该对象包含以下三个属性：

* model - 目标模型名称，比如 '**order**'。
* property - 目标方法名称，比如 '**find'**。
  你也可以指定一个方法名数组，并对数组内所有的方法限定一致的规则。
* accessType - 访问类型，'**EXECUTE'**, '**READ'**, 或 '**WRITE'**

而 ACL 规则就是由一系列的 ACL 记录组成的数组，每个 ACL 记录的属性都必须符合 [Model definition JSON file - ACLs](Model-definition-JSON-file.html#acls) 模型的 JSON 定义文件。
```
- model
- property
- accessType
- principalType
    -  USER: 一个用户 ID
    -  APP: 一个外部应用 ID
    -  ROLE: 一个角色名
- permission
    -  DENY
    -  ALLOW
```

ROLE 角色名可以对应以下三种角色之一：
*  _内建动态角色_, 例如右边数组中任意一项代表一个内建的动态角色 [`$everyone`, `$unauthenticated`, `$authenticated`, `$owner`]
*  [_自定义的静态角色_](Defining-and-using-roles.html#static-roles), 直接映射到 principals
*  [_自定义的动态角色_](Defining-and-using-roles.html#dynamic-roles), 使用自定义的角色分析器来解析

### ACL 规则的优先级

一个单一的模型可能会有多个 ACL 规则约束。这些规则被定义在这个模型自己或其的父模型的 JSON 定义文件 [model definition JSON file](Model-definition-JSON-file.html) 中。
LoopBack 根据 Permission 权限和 Access Type 访问类型的优先级 _叠加覆盖_ 确定最终的 ACL 规则。

Permission 权限优先级如下：
```
1.  DENY
2.  ALLOW
3.  DEFAULT
```

例如，一个操作或用户组同时受一条 `DENY` 规则和一条 `ALLOW` 规则的约束，那么 `DENY` 规则拥有更高的优先级。

Access Type 访问类型的优先级如下：
```
1.  Type 类型 (read, write, replicate, update)
2.  Method name 方法名
3.  Wildcard 通配符
```

通常来说，一个更具体的规则比一个更泛的规则有更高的优先级。
例如，一个禁止已登录认证用户访问的规则比一个禁止所有用户访问的规则有更高的优先级。

LoopBack 通过匹配请求对象和访问规则计算出相应的值用于排序。

(LoopBack sorts multiple rules by the specifics of matching the request against each rule.
It calculates the specifics by checking the access request against each ACL rule by the hierarchical order of attributes.)

At each level, the matching yields three points:
```
 3: exact match
 2: wildcard match (`'*'`)
-1: no match
```

高层级的匹配比底层级的匹配有更高的优先级，例如完全匹配优于通配符匹配。

比如，有以下一个访问请求：

```javascript
{
  model: 'order',
  property: 'find',
  accessType: 'EXECUTE'
}
```

假设我们定义了如下 ACL 规则：

```javascript
[
  // Rule #1
  {
    model: '*',
    property: 'find',
    accessType: 'EXECUTE',
    principalType: 'ROLE',
    principalId: '$authenticated',
    permission: 'ALLOW'
  },
  // Rule #2
  {
    model: 'order',
    property: '*',
    accessType: '*',
    principalType: 'ROLE',
    principalId: '$authenticated',
    permission: 'ALLOW'
  },
  // Rule #3
  {
    model: 'order',
    property: 'find',
    accessType: '*',
    principalType: 'ROLE',
    principalId: '$authenticated',
    permission: 'DENY'
  }
]
```

以上 ACL 规则的优先级顺序 #3, #2, #1。也就是说，这个请求将会被拒绝，因为规则 #3 里的 Permission 权限为 `DENY` 禁止。

## 鉴权作用域

{% include warning.html content="
鉴权作用域的实现还比较初级，并不适合大多数用户，请参与这个讨论 [issue #3339](https://github.com/strongloop/loopback/issues/3339) 帮助我们建立更加高级的 API 特性。
" %}

相关英文文档见 [Controlling Data Access](../../en/lb3/Controlling-data-access.html)

## 调试

在终端指定一个 `DEBUG` 环境变量 `loopback:security:*`，命令行如下：
```
DEBUG=loopback:security:* node .
```
