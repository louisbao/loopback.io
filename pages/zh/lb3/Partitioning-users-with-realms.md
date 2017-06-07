---
title: "使用 realms 域区分用户群体"
lang: zh
layout: page
keywords: LoopBack
tags:
sidebar: zh_lb3_sidebar
permalink: /doc/zh/lb3/Partitioning-users-with-realms.html
summary:
---

{% include see-also.html content="
* [Authentication, authorization, and permissions](Authentication-authorization-and-permissions.html)
* [Third-party login using Passport](Third-party-login-using-Passport.html)
" %}

默认情况下，LoopBack 的 User 用户模型将所有的用户置于同一个空间，而没有根据不同 `客应用` 隔离用户群。
在有些情况下，你有可能需要将用户根据不同 `客应用` 进行分群。LoopBack 通过 _realms_ 实现以下功能：

* 用户和客应用可以被设置为不属于任何域或置于同一个域。
* 将用户和客应用分别置于不同的域，一个用户或客应用可以只属于一个域。一个域也可以有多个用户和多个客应用。
* 一个客应用可以有一个单独域，用户可以通过这个域归属于该客应用。

一个客应用或一个用户在域内有一个唯一的 ID，当客应用或用户注册时可以指定一个域。
在 [`User.login()`](http://apidocs.strongloop.com/loopback/#user-login) 时：

* 允许从用户名和邮件地址前缀中提取 realm 域。

可以通过用户模型中的两项配置控制 realm 域的行为：

* `realmRequired` (Boolean): 默认是 `false`。
* `realmDelimiter` (string): 前缀分隔符 `:` 例如 `myRealm:john` 或 `myRealm:john@sample.com`。如果为空，则不检查和提取前缀。

例如，

{% include code-caption.html content="server/model-config.json" %}
```javascript
"User": {
  "dataSource": "db",
  "options": {
    "realmRequired": true,
    "realmDelimiter": ":"
  }
},
```

当域启用后，你可以在调用 `User.create()` 时提供一个 `realm` 属性值，例如：

```javascript
User.create({
  realm: 'myRealm',
  username: 'john',
  email: 'john@sample.com',
  password: 'my-password'
}, callback);
```

在登录时也可以指定一个 `realm` 域值，

```javascript
User.login({
  realm: 'myRealm',
  username: 'john',
  password: 'my-password'
}, callback);
```

如果指定过 `realmDelimiter` (例如 ":")，则可以在用户名或邮件地址前添加前缀。

```javascript
User.login({
  username: 'myRealm:john',
  password: 'my-password'
}, callback);
```
