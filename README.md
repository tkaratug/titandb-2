# TitanDB - 2
A lightweight and efficient SQL query builder for PHP.

# Installation
The recommended way to install the TitanDB v2 Query Builder is through Composer. Run the following command to install it:
```sh
$ composer require tkaratug/titandb-2
```

# Connection
```php
require 'vendor/autoload.php';
use Titan\DB as DB;

$db = DB::init([
	'db_driver'		=> 'mysql',
	'db_host'		=> 'localhost',
	'db_user'		=> 'sample_db_user',
	'db_pass'		=> '',
	'db_name'		=> 'sample_db_name',
	'db_charset'	=> 'utf8',
	'db_collation'	=> 'utf8_general_ci',
	'db_prefix'	 	=> ''
]);
```

# Contents
* [SELECT](#select)
* [Fetching Multiple Rows](#getAll)
* [Fetching Single Row](#getRow)
* [WHERE](#where)
* [GROUPING WHERE](#whereGroup)
* [JOIN](#join)
* [ORDER BY & LIMIT](#orderby)
* [GROUP BY](#groupby)
* [HAVING](#having)
* [GROUPING HAVING](#havingGroup)
* [LIKE & NOT LIKE](#like)
* [IN & NOT IN](#in)
* [BETWEEN & NOT BETWEEN](#between)
* [INSERT](#insert)
* [UPDATE](#update)
* [DELETE](#delete)
* [LAST INSERT ID](#lastInsertId)
* [ROW COUNT](#numRows)
* [CUSTOM QUERY](#customQuery)

### select
```php
$db->table('test_table')->select('title, content');
// Output: "SELECT title, content FROM test_table"

$db->table('test_table')->select('title as t, content as c');
// Output: "SELECT title as t, content as c FROM test_table"
```

### fetching multiple rows
```php
$db->table('test_table')->getAll();
// Output: "SELECT * FROM test_table"

$db->table('test_table')->select('title, content')->getAll();
// Output: "SELECT title, content FROM test_table"
```

### fetching single row
```php
$db->table('test_table')->where('status', '=', 1)->getRow();
// Output: "SELECT * FROM test_table WHERE status = 1"

$db->table('test_table')->select('title, content')->where('status', '=', 1)->getRow();
// Output: "SELECT title, content FROM test_table WHERE status = 1"
```

### where
```php
$db->table('test_table')
   ->select('title, content')
   ->where('id', '=', 5)
   ->getRow();
// Output: "SELECT title, content FROM test_table WHERE id = 5"

$db->table('test_table')
   ->select('title, content')
   ->where('vote', '>', 20)
   ->where('status', '=', 1)
   ->getAll();
// Output: "SELECT title, content FROM test_table WHERE vote>20 AND status=1"

$db->table('test_table')
   ->select('title, content')
   ->where('vote', '>', 20)
   ->or_where('create_date', '>', date('Y-m-d'))
   ->getAll();
// Output: "SELECT title, content FROM test_table WHERE vote>20 OR create_date>'2017-11-07'"
```

### grouping where
```php
$db->table('test_table')
   ->select('title, content')
   ->where('col_1', '>', 5)
   ->whereGroupStart()
        ->where('col_2', '=', 'val_2')
        ->orWhere('col_2', '=', 'val_3')
   ->whereGroupEnd()
   ->getAll();
// Output: "SELECT title, content FROM test_table WHERE col_1>5 AND (col_2='val_2' OR col_2='val_3')"

$db->table('test_table')
   ->select('title, content')
   ->where('col_1', '>', 5)
   ->whereGroupStart('OR')
        ->where('col_2', '=', 'val_2')
        ->orWhere('col_2', '=', 'val_3')
   ->whereGroupEnd()
   ->getAll();
// Output: "SELECT title, content FROM test_table WHERE col_1>5 OR (col_2='val_2' OR col_2='val_3')"
```

### join
```php
$db->table('users as t1')
   ->leftJoin('comments as t2', 't1.user_id=t2.user_id')
   ->select('t1.username, t2.comment')
   ->getAll();
// Output: "SELECT t1.username, t2.comment FROM users as t1 LEFT JOIN comments as t2 ON t1.user_id=t2.user_id"

$db->table('users as t1')
   ->rightJoin('comments as t2', 't1.user_id=t2.user_id')
   ->select('t1.username, t2.comment')
   ->getAll();
// Output: "SELECT t1.username, t2.comment FROM users as t1 RIGHT JOIN comments as t2 ON t1.user_id=t2.user_id"

$db->table('users as t1')
   ->innerJoin('comments as t2', 't1.user_id=t2.user_id')
   ->select('t1.username, t2.comment')
   ->getAll();
// Output: "SELECT t1.username, t2.comment FROM users as t1 INNER JOIN comments as t2 ON t1.user_id=t2.user_id"

$db->table('users as t1')
   ->outerJoin('comments as t2', 't1.user_id=t2.user_id')
   ->select('t1.username, t2.comment')
   ->getAll();
// Output: "SELECT t1.username, t2.comment FROM users as t1 FULL OUTER JOIN comments as t2 ON t1.user_id=t2.user_id"
```

### order by & limit
```php
$db->table('test_table')->orderBy('id')->getAll();
// Output: "SELECT * FROM test_table ORDER BY id ASC"

$db->table('test_table')->orderBy('id', 'desc')->getAll();
// Output: "SELECT * FROM test_table ORDER BY id DESC"

$db->table('test_table')->orderBy('id', 'desc')->limit(100)->getAll();
// Output: "SELECT * FROM test_table ORDER BY id DESC LIMIT 100"

$db->table('test_table')->order_by('id')->limit(100, 50)->getAll();
// Output: "SELECT * FROM test_table ORDER BY id ASC LIMIT 100, 50"
```

### group by
```php
$db->table('test_table')->select('book, COUNT(*)')->groupBy('book')->getAll();
// Output: "SELECT book, COUNT(*) FROM test_table GROUP BY book"
```

### having
```php
$db->table('test_table')
   ->select('book, COUNT(*)')
   ->groupBy('book')
   ->having('COUNT(*)', '>', 15)
   ->getAll();
// Output: "SELECT book, COUNT(*) FROM test_table GROUP BY book HAVING COUNT(*)>15"

$db->table('test_table')
   ->select('book, COUNT(*)')
   ->groupBy('book')
   ->having('COUNT(*)', '>', 15)
   ->having('COUNT(*)', '<', 50)
   ->getAll();
// Output: "SELECT book, COUNT(*) FROM test_table GROUP BY book HAVING COUNT(*)>15 AND COUNT(*)<50"

$db->table('test_table')
   ->select('book, COUNT(*)')
   ->groupBy('book')
   ->having('COUNT(*)', '>', 15)
   ->orHaving('MAX(price)', '<', 50)
   ->getAll();
// Output: "SELECT book, COUNT(*) GROUP BY book HAVING COUNT(*)>15 OR MAX(price)<50"
```

### like & not like
```php
$db->table('test_table')->select('title, content')->like('title', 'Lorem%')->getAll();
// Output: "SELECT title, content FROM test_table WHERE title LIKE 'Lorem%'"

$db->table('test_table')->select('title, content')->like('title', 'Lorem%')->like('content', '%amet')->getAll();
// Output: "SELECT title, content FROM test_table WHERE title LIKE 'Lorem%' AND content LIKE '%amet'"

$db->table('test_table')->select('title, content')->like('title', 'Lorem%')->orLike('title', 'ipsum%')->getAll();
// Output: "SELECT title, content FROM test_table WHERE title LIKE 'Lorem%' OR title LIKE 'ipsum%'"

$db->table('test_table')->select('title, content')->notLike('title', 'Lorem%')->getAll();
// Output: "SELECT title, content FROM test_table WHERE title NOT LIKE 'Lorem%'"

$db->table('test_table')->select('title, content')->notLike('title', 'Lorem%')->orNotLike('title', 'ipsum%')->getAll();
// Output: "SELECT title, content FROM test_table WHERE title NOT LIKE 'Lorem%' OR title NOT LIKE 'ipsum%'"
```

### in & not in
```php
$db->table('test_table')->select('title, content')->in('categoryId', [1,3,4,6])->getAll();
// Output: "SELECT title, content FROM test_table WHERE categoryId IN(1, 3, 4, 6)"

$db->table('test_table')->select('title, content')->notIn('categoryId', [1,3,4,6])->getAll();
// Output: "SELECT title, content FROM test_table WHERE categoryId NOT IN(1, 3, 4, 6)"
```

### between & not between
```php
$db->table('users')->select('userId, userName')->between('userAge', 20, 30)->getAll();
// Output: "SELECT userId, userName FROM users WHERE userAge BETWEEN 20 AND 30"

$db->table('users')->select('userId, userName')->notBetween('userAge', 20, 30)->getAll();
// Output: "SELECT userId, userName FROM users WHERE userAge NOT BETWEEN 20 AND 30"
```

### insert
```php
$data = [
    'firstName' => 'John',
    'lastName'  => 'Doe',
    'city'      => 34
];
$db->table('users')->insert($data);
// Output: "INSERT INTO users SET firstName='John', lastName='Doe', city=34"
```

### update
```php
$data = [
    'firstName' => 'John',
    'lastName'  => 'Doe',
    'city'      => 34
];
$db->table('users')->where('id', '=', 5)->update($data);
// Output: "UPDATE users SET firstName='John', lastName='Doe', city=34 WHERE id=5"
```

### delete
```php
$db->table('users')->where('id', '=', 5)->delete();
// Output: "DELETE FROM users WHERE id=5"
```

### last insert id
```php
$data = [
    'firstName' => 'John',
    'lastName'  => 'Doe',
    'city'      => 34
];
$db->table('users')->insert($data);
echo $db->lastInsertId();
```

### row count
```php
$db->table('users')->where('age', '>', 20)->getAll();
echo $db->numRows();
```

### last executed query
```php
echo $db->lastQuery();
```

### execute custom query
```php
// Fetching single row
$db->customQuery("SELECT * FROM test_table WHERE id=5")->getRow();

// Fetching multiple rows
$db->customQuery("SELECT * FROM test_table")->getAll();

// Insert
$db->customQuery("INSERT INTO test_table SET col_1='val_1', col_2='val_2'");

// Update
$db->customQuery("UPDATE test_table SET col_1='val_1', col_2='val_2' WHERE id=5");

// Delete
$db->customQuery("DELETE FROM test_table WHERE id=5");

// Execute Stored Procedure
$db->customQuery("CALL procedure_1()");
```