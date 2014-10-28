# 动态查询

动态查询是指由Drupal动态构建的查询，而不是显式提供的查询语句字符串。所有的 `Insert`，`Update`，`Delete` 和 `Merge` 查询必须是动态查询。`Select` 语句可以是静态查询也可以是动态查询。因此，动态查询通常指的是一个动态的 `Select` 查询。

注意：在90%的情况下，你的 SELECT 查询只需要使用静态查询。如果在一个性能有严格要求的页面，基于性能的考虑，你必须使用 `db_query` 和其相关函数，而不要用 `db_select()`。只有当你的查询分成很多部分（例如，基于上下文添加 WHERE 条件）或者查询需要可被修改（例如，[node acess](https://api.drupal.org/api/drupal/modules%21node%21node.module/group/node_access/7)）。注意关于这个有许多的讨论，比如：[#1](https://www.drupal.org/node/1881146) 和 [#2](https://www.drupal.org/node/835068)

Drupal 7 不再有 `db_rewrite_sql`，你必须使用动态查询的机制来完成相同的事情。例如，无论何时查询 node 表，你必须使用 `node_access` 标记，示例代码如下：

``` php
$query = db_select('node', 'n')
  ->addTag('node_access');
```

所有的动态构建查询都是使用一个查询对象构建的，从适当的连接对象实例化。和静态查询一样，在大多数情况下，使用面向过程的封装函数来获得查询对象。然而，查询随后的指令是以对象方法的形式被调用的。

动态 SELECT 查询通过使用 `db_select()` 开始，示例代码如下：

``` php
<?php
$query = db_select('users', 'u', $options);
?>
```

In this case, "users" is the base table for the query; that is, the first table after the FROM statement. Note that it should not have brackets around it. The query builder will handle that automatically. The second parameter is the alias for the table. If not specified, the name of the table is used. The $options array is optional, and is identical to the $options array for static queries.

The value returned by the db_select() call is an object of type SelectQuery. Therefore, the type of value in the $query variable after this call is an object of type SelectQuery. This object has a whole list of methods such as fields(), joins() and group() which can be called to further define the query.

在这个例子中， `users` 是查询的基础表，也就是SQL语句中放在 FROM 后的第一个表。注意，这里不可以用花括号包围表名。查询构建器将会自动帮你处理。第二个参数是表的别名。如果没指定，表名将被使用作为别名。`$options` 数组是可选的，参数格式和作用与静态查询的一样。
