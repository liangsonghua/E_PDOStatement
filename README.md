#PDOStatement::bindParam的一个陷阱
废话不多说, 直接看代码:
```php
$dbh = new PDO('mysql:host=localhost;dbname=test', "test");
 
$query = <<<QUERY
  INSERT INTO `user` (`username`, `password`) VALUES (:username, :password);
QUERY;
$statement = $dbh->prepare($query);
 
$bind_params = array(':username' => "laruence", ':password' => "weibo");
foreach( $bind_params as $key => $value ){
    $statement->bindParam($key, $value);
}
$statement->execute();

```

 请问, 最终执行的SQL语句是什么, 上面的代码是否有什么问题?

Okey, 我想大部分同学会认为, 最终执行的SQL是:
```sql
INSERT INTO `user` (`username`, `password`) VALUES ("laruence", "weibo");
```

但是, 可惜的是, 你错了, 最终执行的SQL是:

```sql
IINSERT INTO `user` (`username`, `password`) VALUES ("weibo", "weibo");
```

 究其原因, 也就是bindParam和bindValue的不同之处, bindParam要求第二个参数是一个引用变量(reference).

让我们把上面的代码的foreach拆开, 也就是这个foreach:

```php
foreach( $bind_params as $key => $value ){
    $statement->bindParam($key, $value);
}
//第一次循环
$value = $bind_params[":username"];
$statement->bindParam(":username", &$value); //此时, :username是对$value变量的引用
 
//第二次循环
$value = $bind_params[":password"]; //oops! $value被覆盖成了:password的值
$statement->bindParam(":password", &$value);
```

##建议

 所以, 在使用bindParam的时候, 尤其要注意和foreach联合使用的这个陷阱. 那么正确的作法呢?

1. 不要使用foreach, 而是手动赋值

```php
<?php
$statement->bindParam(":username", $bind_params[":username"]); //$value是引用变量了
$statement->bindParam(":password", $bind_params[":password"]);
```

 2. 使用bindValue代替bindParam, 或者直接在execute中传递整个参数数组.

3. 使用foreach和reference(不推荐)
