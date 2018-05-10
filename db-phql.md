# Phalcon查询语言(Phalcon Query Language)
Phalcon查询语言，简称PhalconQL或PHQL，是一种面向对象的高级SQL语言，允许用标准化的SQL编写。PHQL实现了把操作语句解析为RDBMS目标语言的解析器(C语言编写)。

为了达到最佳性能，Phalcon提供了与SQLite相同的解析器，其线程安全，内存占用极低。

解析器先检查传递的PHQL语句的语法，然后构建中间语句，最后将其转换为RDBMS对应的SQL语句。

PHQL实现了一系列功能，可以更安全的操作数据库。

- 参数绑定是PHQL功能之一，使代码更安全
- PHQL每次只允许执行一条SQL语句，以防SQL注入
- PHQL会忽略所有SQL注入中常用的SQL注释
- PHQL只允许数据操作语句，避免错误的或未经授权的更改、删除数据库和表
- PHQL实现了高级抽象接口，允许以模型方式操作表，以类属性方式操作表字段

## 使用示例(Usage Example)

为了更好的解释PHQL工作原理，请参考下例。有`Cars`和`Brands`两个模型：
```php
<?php

use Phalcon\Mvc\Model;

class Cars extends Model
{
    public $id;

    public $name;

    public $brand_id;

    public $price;

    public $year;

    public $style;

    // 模型Cars映射sample_cars表
    public function getSource()
    {
        return 'sample_cars';
    }

    // 一辆车属于一个品牌，但一个品牌有多辆车
    public function initialize()
    {
        $this->belongsTo('brand_id', 'Brands', 'id');
    }
}
```
每辆车都属于一个品牌，每个品牌有多辆车：
```php
<?php

use Phalcon\Mvc\Model;

class Brands extends Model
{
    public $id;

    public $name;

    // 模型Brands映射表'sample_brands'
    public function getSource()
    {
        return 'sample_brands';
    }

    // 一个品牌有多辆车
    public function initialize()
    {
        $this->hasMany('id', 'Cars', 'brand_id');
    }
}
```
## 创建PHQL查询(Creating PHQL Queries)
实例化`Phalcon\Mvc\Model\Query`类即可创建PHQL查询：
```php
<?php

use Phalcon\Mvc\Model\Query;

// 实例化Query
$query = new Query(
    "SELECT * FROM Cars",
    $this->getDI()
);

// 执行查询，返回结果(如果有的话)
$cars = $query->execute();
```
控制器或视图中，使用`Phalcon\Mvc\Model\Manager`可以很容易的创建、执行PHQL查询：
```php
<?php

// 执行简单查询
$query = $this->modelsManager->createQuery("SELECT * FROM Cars");
$cars  = $query->execute();

// 使用参数绑定
$query = $this->modelsManager->createQuery("SELECT * FROM Cars WHERE name = :name:");
$cars  = $query->execute(
    [
        'name' => 'Audi',
    ]
);
```
或者直接执行查询：
```php
<?php

// 执行简单查询
$cars = $this->modelsManager->executeQuery(
    "SELECT * FROM Cars"
);

// 使用参数绑定
$cars = $this->modelsManager->executeQuery(
    "SELECT * FROM Cars WHERE name = :name:",
    [
        'name' => 'Audi',
    ]
);
```
## 查询记录(Selecting Records)
PHQL允许使用我们熟知的SELECT语句查询记录，使用模型名字代替表名：
```php
<?php

$query = $manager->createQuery(
    "SELECT * FROM Cars ORDER BY Cars.name"
);

$query = $manager->createQuery(
    "SELECT Cars.name FROM Cars ORDER BY Cars.name"
);
```
允许带命名空间的模型名：
```php
<?php

$phql  = "SELECT * FROM Formula\Cars ORDER BY Formula\Cars.name";
$query = $manager->createQuery($phql);

$phql  = "SELECT Formula\Cars.name FROM Formula\Cars ORDER BY Formula\Cars.name";
$query = $manager->createQuery($phql);

$phql  = "SELECT c.name FROM Formula\Cars c ORDER BY c.name";
$query = $manager->createQuery($phql);
```
PHQL支持大部分标准SQL语法，非标准的SQL语法也同样支持，如LIMIT：
```php
<?php

$phql = "SELECT c.name FROM Cars AS c WHERE c.brand_id = 21 ORDER BY c.name LIMIT 100";

$query = $manager->createQuery($phql);
```
### 结果集类型(Result Types)
结果集类型根据我们查询字段的不同而不同，如果检索单个完整对象，则返回`Phalcon\Mvc\Model\Resultset\Simple`对象。这种结果集是一组完整的模型对象：
```php
<?php

$phql = "SELECT c.* FROM Cars AS c ORDER BY c.name";

$cars = $manager->executeQuery($phql);

foreach ($cars as $car) {
    echo 'Name: ', $car->name, "\n";
}
```
下面这种方式也一样：
```php
<?php

$cars = Cars::find(
    [
        'order' => 'name',
    ]
);

foreach ($cars as $car) {
    echo 'Name: ', $car->name, "\n";
}
```
完整模型对象中的数据能够被修改，并重新保存到数据库中，因为它们代表关联表的完整记录。下面这种查询方式不会返回完整模型对象：
```php
<?php

$phql = "SELECT c.id, c.name FROM Cars AS c ORDER BY c.name";

$cars = $manager->executeQuery($phql);

foreach ($cars as $car) {
    echo 'Name: ', $car->name, "\n";
}
```
我们仅仅查询了表中的某些字段，虽然返回的结果集仍然是`Phalcon\Mvc\Model\Resultset\Simple`对象，但不能当成完整模型对象。该对象的每个成员都是一个包含所查询字段的标准对象。

这些不表示完整对象的值就是我们所说的标量，PHQL允许查询所有类型的标量：字段，函数，字面两，表达式等：
```php
<?php

$phql = "SELECT CONCAT(c.id, ' ', c.name) AS id_name FROM Cars AS c ORDER BY c.name";

$cars = $manager->execute($phql);

foreach ($cars as $car) {
    echo $car->id_name, "\n";
}
```
我们可以查询完整对象或标量，也可以同时查询它们：
```php
<?php

$phql = "SELECT c.price*0.16 AS taxes, c.* FROM Cars AS c ORDER BY c.name";

$result = $manager->executeQuery($phql);
```
这种情况下的结果集是一个`Phalcon\Mvc\Model\Resultset\Complex`对象，可以同时访问完整对象和标量：
```php
<?php

foreach ($result as $row) {
    echo 'Name: ', $row->cars->name, "\n";
    echo 'Price: ', $row->cars->price, "\n";
    echo 'Taxes: ', $row->taxes, "\n";
}
```
### 连接(Joins)
使用PHQL可以很容易的从多个模型请求记录，支持大部分的JOIN方式。我们在模型中定义关系之后，PHQL会自动添加这些条件：
```php
<?php

$phql = "SELECT Cars.name AS car_name, Brands.name AS brand_name FROM Cars JOIN Brands";

$rows = $manager->executeQuery($phql);

foreach ($rows as $row) {
    echo $row->car_name, "\n";
    echo $row->brand_name, "\n";
}
```
默认使用INNER JOIN，可以指定JOIN类型：
```php
<?php

$phql = "SELECT Cars.*, Brands.* FROM Cars INNER JOIN Brands";
$rows = $manager->executeQuery($phql);

$phql = "SELECT Cars.*, Brands.* FROM Cars LEFT JOIN Brands";
$rows = $manager->executeQuery($phql);

$phql = "SELECT Cars.*, Brands.* FROM Cars LEFT OUTER JOIN Brands";
$rows = $manager->executeQuery($phql);

$phql = "SELECT Cars.*, Brands.* FROM Cars CROSS JOIN Brands";
$rows = $manager->executeQuery($phql);
```
也可以手动设置JOIN条件：
```php
<?php

$phql = "SELECT Cars.*, Brands.* FROM Cars INNER JOIN Brands ON Brands.id = Cars.brands_id";

$rows = $manager->executeQuery($phql);
```
如果查询中为模型定义别名，则将使用别名为结果集中的每一条记录命名：
```php
<?php

$phql = "SELECT c.*, b.* FROM Cars c, Brands b WHERE b.id = c.brands_id";

$rows = $manager->executeQuery($phql);

foreach ($rows as $row) {
    echo 'Car: ', $row->c->name, "\n";
    echo 'Brand: ', $row->b->name, "\n";
}
```
如果连接模型与`from`之后的模型具有多对多关系时，中间模型将隐式的添加到查询中：
```php
<?php

$phql = "SELECT Artists.name, Songs.name FROM Artists JOIN Songs WHERE Artists.genre = 'Trip-Hop'";

$result = $this->modelsManager->executeQuery($phql);
```
上述代码在MySQL中执行下列SQL：
```sql
SELECT `artists`.`name`, `songs`.`name` FROM `artists`
INNER JOIN `albums` ON `albums`.`artists_id` = `artists`.`id`
INNER JOIN 'songs' ON `albums`.`songs_id` = `songs`.`id`
WHERE `artists`.`genre` = 'Trip-Hop'
```
### 聚合(Aggregations)
下面例子展示了PHQL中如何使用聚合：
```php
<?php

// 所有汽车的总价值
$phql = "SELECT SUM(price) AS summatory FROM Cars";
$row  = $manager->executeQuery($phql)->getFirst();
echo $row['summatory'];

// 每个品牌下的汽车总数
$phql = "SELECT Cars.brand_id, COUNT(*) FROM Cars GROUP BY Cars.brand_id";
$rows = $manager->executeQuery($phql);
foreach ($rows as $row) {
    echo $row->brand_id, ' ', $row['1'], "\n";
}

// 每个品牌下的汽车总数
$phql = "SELECT Brands.name, COUNT(*) FROM Cars JOIN Brands GROUP BY 1";
$rows = $manager->executeQuery($phql);
foreach ($rows as $row) {
    echo $row->name, ' ', $row['1'], "\n";
}

$phql = "SELECT MAX(price) AS maximum, MIN(price) AS minimum FROM Cars";
$rows = $manager->executeQuery($phql);
foreach ($rows as $row) {
    echo $row['maximum'], ' ', $row['minimum'], "\n";
}

// 统计品牌数量
$phql = "SELECT COUNT(DISTINCT brand_id) AS brandId FROM Cars";
$rows = $manager->executeQuery($phql);
foreach ($rows as $row) {
    echo $row->brandId, "\n";
}
```
### 条件(Conditions)
条件能让我们过滤想要查询的记录，`WHERE`子句允许这样:
```php
<?php

// 简单条件
$phql = "SELECT * FROM Cars WHERE Cars.name = 'Lamborghini Espada'";
$cars = $manager->executeQuery($phql);

$phql = "SELECT * FROM Cars WHERE Cars.price > 10000";
$cars = $manager->executeQuery($phql);

$phql = "SELECT * FROM Cars WHERE TRIM(Cars.name) = 'Audi R8'";
$cars = $manager->executeQuery($phql);

$phql = "SELECT * FROM Cars WHERE Cars.name LIKE 'Ferrari%'";
$cars = $manager->executeQuery($phql);

$phql = "SELECT * FROM Cars WHERE Cars.name NOT LIKE 'Ferrari%'";
$cars = $manager->executeQuery($phql);

$phql = "SELECT * FROM Cars WHERE Cars.price IS NULL";
$cars = $manager->executeQuery($phql);

$phql = "SELECT * FROM Cars WHERE Cars.id IN (120, 121, 122)";
$cars = $manager->executeQuery($phql);

$phql = "SELECT * FROM Cars WHERE Cars.id NOT IN(430, 431)";
$cars = $manager->executeQuery($phql);

$phql = "SELECT * FROM Cars WHERE Cars.id BETWEEN 1 AND 100";
$cars = $manager->executeQuery($phql);
```
此外，作为PHQL的一部分，参数绑定会自动转义输入数据，安全性更高：
```php
<?php

$phql = "SELECT * FROM Cars WHERE Cars.name = :name:";
$cars = $manager->executeQuery(
    $phql,
    [
        'name' => 'Lamborghini Espada',
    ]
);

$phql = "SELECT * FROM Cars WHERE Cars.name = ?0";
$cars = $manager->executeQuery(
    $phql,
    [
        0 => 'Lamborghini Espada',
    ]
);
```
## 插入数据(Inserting Data)
通过PHQL，可以使用我们非常熟悉的INSERT语句插入数据：
```php
<?php

// 插入数据，不指定字段
$phql = "INSERT INTO Cars VALUES (NULL, 'Lamborghini Espada', 7, 10000.00, 1969, 'Grand Tourer')";
$manager->executeQuery($phql);

// 插入数据，指定字段
$phql = "INSERT INTO Cars (name, brand_id, year, style) VALUES ('Lamborghini Espada', 7, 1969, 'Grand Tourer')";
$manager->executeQuery($phql);

// 插入数据，使用占位符
$phql = "INSERT INTO Cars (name, brand_id, year, style) VALUES (:name:, :brand_id:, :year:, :style:)";
$manager->executeQuery(
    $phql,
    [
        'name'     => 'Lamborghini Espada',
        'brand_id' => 7,
        'year'     => 1969,
        'style'    => 'Grand Tourer',
    ]
);
```
Phalcon不只是单纯的将PHQL语句转化成SQL，模型中定义的所有事件和业务规则都会执行，就像我们手动创建对象那样。我们为模型Cars创建一条规则，车的价格不能低于$ 10,000：
```php
<?php

use Phalcon\Mvc\Model;
use Phalcon\Mvc\Model\Message;

class Cars extends Model
{
    public function beforeCreate()
    {
        if ($this->price < 10000) {
            $this->appendMessage(
                new Message('A car cannot cost less than $ 10,000')
            );

            return false;
        }
    }
}
```
如果我们在模型Cars中执行下面的`INSERT`语句，操作将会失败，因为price不满足我们制定的规则。通过检查插入状态，我们可以打印任何内部生成的验证消息：
```php
<?php

$phql = "INSERT INTO Cars VALUES (NULL, 'Nissan Versa', 7, 9999.00, 2015, 'Sedan')";

$result = $manager->executeQuery($phql);

if ($result->success() === false) {
    foreach ($result->getMessages() as $message) {
        echo $message->getMessage();
    }
}
```
## 更新数据(Updating Data)
更新记录与插入记录非常相似，更新记录使用`UPDATE`命令。更新记录时，将为每条记录执行与更新操作相关的事件。
```php
<?php

// 更新一个字段
$phql = "UPDATE Cars SET price = 15000.00 WHERE id = 101";
$manager->executeQuery($phql);

// 更新多个字段
$phql = "UPDATE Cars SET price = 15000.00, type = 'Sedan' WHERE id = 101";
$manager->executeQuery($phql);

// 更新多条记录
$phql = "UPDATE Cars SET price = 7000.00, type = 'Sedan' WHERE brands_id > 5";
$manager->executeQuery($phql);

// 使用占位符
$phql = "UPDATE Cars SET price = ?0, type = ?1 WHERE brands_id > ?2";
$manager->executeQuery(
    $phql,
    [
        0 => 7000.00,
        1 => 'Sedan',
        2 => 5,
    ]
);
```
`UPDATE`语句执行更新分两步进行：

- 首先，如果`UPDATE`包含`WHERE`子句，将检索符合条件的所有对象
- 其次，基于查询对象更新字段并保存

这种操作方式允许事件、虚拟外键和验证参与更新过程。
```php
<?php

$phql = "UPDATE Cars SET price = 15000.00 WHERE id > 101";

$result = $manager->executeQuery($phql);

if ($result->success() === false) {
    $messages = $result->getMessages();

    foreach ($messages as $message) {
        echo $message->getMessage();
    }
}
```
上面代码相当于：
```php
<?php

$messages = null;

$process = function () use (&$messages) {
    $cars = Cars::find('id > 101');

    foreach ($cars as $car) {
        $car->price = 15000;

        if ($car->save() === false) {
            $messages = $car->getMessages();

            return false;
        }
    }

    return true;
};

$success = $process();
```
## 删除数据(Deleting Data)
删除记录时，与删除操作相关的事件将逐一执行：
```php
<?php

// 删除一条记录
$phql = "DELETE FROM Cars WHERE id = 101";
$manager->executeQuery($phql);

// 删除多条记录
$phql = "DELETE FROM Cars WHERE id > 100";
$manager->executeQuery($phql);

// 使用占位符
$phql = "DELETE FROM Cars WHERE id BETWEEN :initial: AND :final:";
$manager->executeQuery(
    $phql,
    [
        'initial' => 1,
        'final'   => 100,
    ]
);
```
和`UPDATE`一样，`DELETE`操作也分两步执行，要检查删除操作是否产生验证消息，你可以检查返回的状态：
```php
<?php

// 删除多条记录
$phql = "DELETE FROM Cars WHERE id > 100";

$result = $manager->executeQuery($phql);

if ($result->success() === false) {
    $messages = $result->getMessages();

    foreach ($messages as $message) {
        echo $message->getMessage();
    }
}
```
## 使用查询构造器创建查询(Creating queries using the Query Builder)
查询构造器可用于创建PHQL查询，无需编写PHQL语句：
```php
<?php

// 获取所有记录
$robots = $this->modelsManager->createBuilder()
    ->from('Robots')
    ->join('RobotsParts')
    ->orderBy('Robots.name')
    ->getQuery()
    ->execute();

// 获取第一条记录
$robots = $this->modelsManager->createBuilder()
    ->from('Robots')
    ->join('RobotsParts')
    ->orderBy('Robots.name')
    ->getQuery()
    ->getSingleResult();
```
同下列操作：
```php
<?php

$phql = "SELECT Robots.* FROM Robots JOIN RobotsParts p ORDER BY Robots.name LIMIT 20";

$result = $manager->executeQuery($phql);
```
查询构造器更多示例：
```php
<?php

// "SELECT Robots.* FROM Robots";
$builder->from('Robots');

// "SELECT Robots.*, RobotsParts.* FROM Robots, RobotsParts";
$builder->from(
    [
        'Robots',
        'RobotsParts',
    ]
);

// "SELECT * FROM Robots";
$phql = $builder->columns('*')
    ->from('Robots');

// "SELECT id FROM Robots";
$builder->columns('id')
    ->from('Robots');

// "SELECT id, name FROM Robots";
$builder->columns(['id', 'name'])
    ->from('Robots');

// "SELECT Robots.* FROM Robots WHERE Robots.name = 'Voltron'";
$builder->from('Robots')
    ->where('Robots.name = "Voltron"');

// "SELECT Robots.* FROM Robots WHERE Robots.id = 100";
$builder->from('Robots')
    ->where(100);

// "SELECT Robots.* FROM Robots WHERE Robots.type = 'virtual' AND Robots.id > 50";
$builder->from('Robots')
    ->where('type = "virtual"')
    ->andWhere('id > 50');

// "SELECT Robots.* FROM Robots WHERE Robots.type = 'virtual' OR Robots.id > 50";
$builder->from('Robots')
    ->where('type = "virtual"')
    ->orWhere('id > 50');

// "SELECT Robots.* FROM Robots GROUP BY Robots.name";
$builder->from('Robots')
    ->groupBy('Robots.name');

// "SELECT Robots.* FROM Robots GROUP BY Robots.name, Robots.id";
$builder->from('Robots')
    ->groupBy(['Robots.name', 'Robots.id']);

// "SELECT Robots.name SUM(Robots.price) FROM Robots GROUP BY Robots.name";
$builder->columns(['Robots.name', 'SUM(Robots.price)'])
    ->from('Robots')
    ->groupBy('Robots.name');

// "SELECT Robots.name, SUM(Robots.price) FROM Robots GROUP BY Robots.name HAVING SUM(Robots.price) > 1000";
$builder->columns(['Robots.name', 'SUM(Robots.price)'])
    ->from('Robots')
    ->groupBy('Robots.name')
    ->having('SUM(Robots.price) > 1000');

// "SELECT Robots.* FROM Robots JOIN RobotsParts";
$builder->from('Robots')
    ->join('RobotsParts');

// "SELECT Robots.* FROM Robots JOIN RobotsParts AS p";
$builder->from('Robots')
    ->join('RobotsParts', null, 'p');

// "SELECT Robots.* FROM Robots JOIN RobotsParts ON Robots.id = RobotsParts.robots_id AS p";
$builder->from('Robots')
    ->join('RobotsParts', 'Robots.id = RobotsParts.robots_id', 'p');

// "SELECT Robots.* FROM robots JOIN RobotsParts ON Robots.id = RobotsParts.robots_id AS p JOIN Parts ON Parts.id = RobotsParts.parts_id AS t";
$builder->from('Robots')
    ->join('RobotsParts', 'Robots.id = RobotsParts.robots_id', 'p')
    ->join('RobotsParts', 'Parts.id = RobotsParts.parts_id', 't');

// "SELECT r.* FROM Robots AS r";
$builder->addFrom('Robots', 'r');

// "SELECT Robots.*, p.* FROM Robots, Parts AS p";
$builder->from('Robots')
    ->addFrom('Parts', 'p');

// "SELECT r.*, p.* FROM Robots AS r, Parts AS p";
$builder->from(['r' => 'Robots'])
    ->addFrom('Parts', 'p');

// "SELECT r.*, p.* FROM Robots AS r, Parts AS p";
$builder->from(['r' => 'Robots', 'p' => 'Parts']);

// "SELECT Robots.* FROM Robots LIMIT 10";
$builder->from('Robots')
    ->limit(10);

// "SELECT Robots.* FROM Robots LIMIT 10 OFFSET 5";
$builder->from('Robots')
    ->limit(10, 5);

// "SELECT Robots.* FROM Robots WHERE id BETWEEN 1 AND 100";
$builder->from('Robots')
    ->betweenWhere('id', 1, 10);

// "SELECT Robots.* FROM Robots WHERE id IN (1, 2, 3)";
$builder->from('Robots')
    ->inWhere('id', [1, 2, 3]);

// "SELECT Robots.* FROM Robots WHERE id NOT IN (1, 2, 3)";
$builder->from('Robots')
    ->notInWhere('id', [1, 2, 3]);

// "SELECT Robots.* FROM Robots WHERE name LIKE '%Art%'";
$builder->from('Robots')
    ->where('name LIKE :name:', ['name' => '%' . $name . '%']);

// "SELECT r.* FROM Store\Robots WHERE r.name LIKE '%Art%'";
$builder->from(['r' => 'Store\Robots'])
    ->where('r.name LIKE :name:', ['name' => '%' . $name . '%']);
```
### 参数绑定(Bound Parameters)
查询构造器中的参数绑定可以在查询构建时设置，也可以在查询执行时设置：
```php
<?php

// 构建查询时传递参数
$robots = $this->modelsManager->createBuilder()
    ->from('Robots')
    ->where('name = :name:', ['name' => $name])
    ->andWhere('type = :type:', ['type' => $type])
    ->getQuery()
    ->execute();

// 执行查询时传递参数
$robots = $this->modelsManager->createBuilder()
    ->from('Robots')
    ->where('name = :name:')
    ->andWhere('type = :type:')
    ->getQuery()
    ->execute(['name' => $name, 'type' => $type]);
```
## 禁用字面量(Disallow literals in PHQL)
PHQL中可以禁用字面量，这意味着如果禁用开启，则不能在PHQL语句中直接使用PHP字符串、数字和布尔值。如果在PHQL语句中嵌入外部数据，可能导致潜在的注入攻击：
```php
<?php

$login  = 'voltron';
$phql   = "SELECT * FROM Models\Users WHERE login = '{$login}'";
$result = $manager->executeQuery($phql);
```
如果`$login`的值为' OR ' ' = ' ，将产生如下PHQL语句：
```sql
SELECT * FROM Models\Users WHERE login = '' OR '' = '';
```
无论存储在数据库中的`login`是何值，条件总是`true`。

如果字面量被禁用，在PHQL中使用PHP字面量会抛出异常，以强制开发者使用参数绑定。上面的查询这样写更安全：
```php
<?php

$type   = 'virtual';
$phql   = "SELECT Robots.* FROM Robots WHERE Robots.type = :type:";
$result = $manager->executeQuery(
    $phql,
    [
        'type' => $type,
    ]
);
```
可以通过以下方式禁用字面量：
```php
<?php

use Phalcon\Mvc\Model;

Model::setup(
    ['phqlLiterals' => false]
);
```
无论字面量是否禁用，参数绑定都可以正常使用。禁用只是开发人员能够在web应用中采取的一项安全策略。
## 转义保留字(Escaping Reserved Words)
PHQL有一些保留字，如果想将保留字作为模型名或字段名使用，则需要使用转义分隔符`[`和`]`来转义关键字：
```php
<?php

$phql   = "SELECT * FROM [Update]";
$result = $manager->executeQuery($phql);

$phql   = "SELECT id, [Like] FROM Posts";
$result = $manager->executeQuery($phql);
```
## PHQL生命周期(PHQL Lifecycle)
作为高级语言，PHQL赋予了开发者个性化定制的能力，以满足不同的需求。以下是PHQL语句的生命周期：

- PHQL被解析并转换为独立于数据库SQL之外的中间表示(IR)
- 根据模型对应的数据库系统，IR被转换为有效的SQL
- PHQL语句被解析并保存在内存中，再次执行相同语句时速度会更快
## 使用原生SQL(Using Raw SQL)
某些数据库系统可能会提供PHQL不支持的特殊SQL扩展，这种情况适合使用原生SQL：
```php
<?php

use Phalcon\Mvc\Model;
use Phalcon\Mvc\Model\Resultset\Simple as Resultset;

class Robots extends Model
{
    public static function findByCreateInterval()
    {
        // 原生SQL
        $sql = "SELECT * FROM robots WHERE id > 0";

        // 模型
        $robot = new Robots();

        // 执行查询
        return new Resultset(
            null,
            $robot,
            $robot->getReadConnection()->query($sql)
        );
    }
}
```
如果原生SQL查询在应用中很普遍，可以在模型中添加通用方法：
```php
<?php

use Phalcon\Mvc\Model;
use Phalcon\Mvc\Model\Resultset\Simple as Resultset;

class Robots extends Model
{
    public static function findByRawSql($conditions, $params = null)
    {
        // 原生SQL
        $sql = "SELECT * FROM robots WHERE {$conditions}";

        // 模型
        $robot = new Robots();

        // 执行查询
        return new Resultset(
            null,
            $robot,
            $robot->getReadConnection()->query($sql)
        );
    }
}
```
上述`findByRawSQL`可以如下使用：
```php
<?php

$robots = Robots::findByRawSql(
    'id > ?',
    [
        10,
    ]
);
```
## 注意事项(Troubleshooting)
PHQL中的一些注意事项：

- 类名称区分大小写，如果定义类时名称和创建时的名称不一致，在大小写敏感的操作系统(如linux)中将导致不可预知行为
- 为保证参数绑定成功，连接数据库时必须指定正确的字符集
- 指定别名的类不能用完整命名空间替换，因为这项操作发生在PHP代码中，而非PHQL语句里
- 如果字段使用别名，应避免别名和字段名相同，不然查询解析器容易混淆。