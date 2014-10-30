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

在这个例子中， `users` 是查询的基础表，也就是SQL语句中放在 FROM 后的第一个表。注意，这里不可以用花括号包围表名。查询构建器将会自动帮你处理。第二个参数是表的别名。如果没指定，表名将被使用作为别名。`$options` 数组是可选的，参数格式和作用与静态查询的一样。

`db_select()` 函数调用的返回值是一个 `SelectQuery` 类的对象。因此，`$query` 变量也是 `SelectQuery` 类的对象。这个对象有多个可用方法，例如 `fields()`, `joins()` 和 `group()`，他们被用来进一步的定义查询。

动态查询可以非常简单，也可能很复杂。下面我们将看看构建一个简单查询的各个部分，在接下来的问当中我们将看到更多高级技巧，例如joins。

## 主要部分

Here is a relatively simple query of the users table.

Let's say we want to create a dynamic query which is roughly equivalent to the following static query:

这是一个相对简单的，对 users 表的查询。

我们的目的是构建一个动态查询，其等价于下面的静态查询：

``` php
<?php
$result = db_query("SELECT uid, name, status, created, access FROM {users} u WHERE uid <> 0 LIMIT 50 OFFSET 0");
?>
```

动态查询可以被写成下面的样子：

``` php
<?php
// Create an object of type SelectQuery
$query = db_select('users', 'u');
// Add extra detail to this query object: a condition, fields and a range
$query->condition('u.uid', 0, '<>');
$query->fields('u', array('uid', 'name', 'status', 'created', 'access'));
$query->range(0, 50);
?>
```

This is often written using the shorthand syntax used to call multiple methods at the same time on the $query object. The code above would therefore often be written as follows:

通常我们使用其简洁的链式调用语法对 `$query` 对象同时调用多个方法。上面的代码通常被写成下面的样子：

``` php
<?php
// Create an object of type SelectQuery
$query = db_select('users', 'u');
// Add extra detail to this query object: a condition, fields and a range
$query
  ->condition('u.uid', 0, '<>')
  ->fields('u', array('uid', 'name', 'status', 'created', 'access'))
  ->range(0, 50);
?>
```

确实，代码还可以而且经常被写成一步到位的，从 `db_select()` 开始直接一直调用生成查询结果，即：

```
<?php
// Create an object of type SelectQuery and directly
// add extra detail to this query object: a condition, fields and a range
$query = db_select('users', 'u')
  ->condition('u.uid', 0, '<>')
  ->fields('u', array('uid', 'name', 'status', 'created', 'access'))
  ->range(0, 50);
?>
```

这是一个用于用户管理页的查询的简化形式，更复杂的形式可以参考[这篇文档](https://api.drupal.org/api/drupal/modules%21user%21user.admin.inc/function/user_admin_account/7)。

## 执行查询

The execute() method will return a result set / statement object that is identical to that returned by db_query(), and it may be iterated or fetched in the exact same way:

<?php
$result = $query->execute();
foreach ($result as $record) {
  // Do something with each $record
}
?>
Note: Be careful when using the following methods with a multi-column, dynamic query:
一旦查询被构建，调用 `execute()` 方法编译和执行查询。

``` php
<?php
$result = $query->execute();
?>
```

The execute() method will return a result set / statement object that is identical to that returned by db_query(), and it may be iterated or fetched in the exact same way:

execute()方法可以返回一个结果集对象，等价于使用 `db_query()` 
