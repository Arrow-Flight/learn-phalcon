# 数据库抽象层(Database Abstraction Layer)
`Phalcon\Db`是`Phalcon\Mvc\Model`底层组件，由它驱动框架中的模型层。它完全由C语言编写，是一个独立的数据库高级抽象层。

与传统模型相比，该组件允许更底层的数据库操作。
## 数据库适配器(Database Adapters)
该组件使用适配器来封装特定的数据库操作。Phalcon使用PDO连接数据库，支持下列数据库引擎：
<table>
    <thead>
        <tr>
            <th>类</th>
            <th>说明</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <code>Phalcon\\Db\\Adapter\\Pdo\\Mysql</code>
            </td>
            <td>世界上最流行的关系型数据库系统(RDBMS)，作为服务器运行，支持多用户、多数据库访问</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\\Db\\Adapter\\Pdo\\Postgresql</code>
            </td>
            <td>Postgresql是一个强大的开源关系数据库系统，超过15年的发展和通过验证的架构，为其赢得了正确、可靠、数据完整的良好声誉</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\\Db\\Adapter\\Pdo\\Sqlite</code>
            </td>
            <td>SQLite是一个实现自包含、无服务、零配置的事务型数据库</td>
        </tr>
    </tbody>
</table>

### 工厂类
使用适配器选项加载PDO：
```php
<?php

use Phalcon\Db\Adapter\Pdo\Factory;

$options = [
    'host'     => 'localhost',
    'dbname'   => 'blog',
    'port'     => 3306,
    'username' => 'sigma',
    'password' => 'secret',
    'adapter'  => 'mysql',
];

$db = Factory::load($options);
```
### 自定义适配器(Implementing your own adapters)
创建自定义数据库适配器或扩展现有适配器，必须实现`Phalcon\Db\AdapterInterface`接口。
## 数据库语言(Database Dialects)
phalcon语言封装了每个数据库的具体操作，为适配器提供通用方法和SQL生成器。
<table>
    <thead>
        <tr>
            <th>类</th>
            <td>说明</td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <code>Phalcon\\Db\\Dialect\\Mysql</code>
            </td>
            <td>MySQL特定语言</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\\Db\\Dialect\\Postgresql</code>
            </td>
            <td>Postgresql特定语言</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\\Db\\Dialect\\Sqlite</code>
            </td>
            <td>SQLite特定语言</td>
        </tr>
    </tbody>
</table>

### 自定义语言(Implementing your own dialects)
创建自定义数据库语言或扩展现有语言，必须实现`Phalcon\Db\DialectInterface`接口。
## 连接数据库(Connection to Databases)
建立数据库连接，必须实例化适配器类，它只接收一个包含连接参数的数组。下面例子展示了如何传递必选参数和可选参数来建立数据库连接：
```php
<?php

// 必选参数
$config = [
    'host'     => '127.0.0.1',
    'username' => 'mike',
    'password' => 'sigma',
    'dbname'   => 'test_db',
];

// 可选参数
$config['persistent'] = false;

// 建立连接
$connection = new \Phalcon\Db\Adapter\Pdo\Mysql($config);
```
```php
<?php

// 必选参数
$config = [
    'host'     => 'localhost',
    'username' => 'postgres',
    'password' => 'secret1',
    'dbname'   => 'template',
];

// 可选参数
$config['schema'] = 'public';

// 建立连接
$connection = new \Phalcon\Db\Adapte\Pdo\Postgresql($config);
```
## 设置额外的PDO选项(Setting up additional PDO options)
你可以在建立连接的时候,传递`options`参数设置PDO：
```php
<?php

// 使用PDO选项建立连接
$connection = new \Phalcon\Db\Adapter\Pdo\Mysql(
    [
        'host'     => 'localhost',
        'username' => 'root',
        'password' => 'sigma',
        'dbname'   => 'test_db',
        'options'  => [
            PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES UTF8',
            PDO::ATTR_CASE               => PDO::CASE_LOWER,
        ],
    ]
);
```
## 使用工厂类连接数据库(Connecting using Factory)
你可以使用一个简单的ini文件来配置/连接数据库。
```ini
[database]
host     = TEST_DB_MYSQL_HOST
username = TEST_DB_MYSQL_USER
password = TEST_DB_MYSQL_PASSWD
dbname   = TEST_DB_MYSQL_NAME
port     = TEST_DB_MYSQL_PORT
charset  = TEST_DB_MYSQL_CHARSET
adapter  = mysql
```
```php
<?php

use Phalcon\Config\Adapter\Ini;
use Phalcon\Db\Adapter\Pdo\Factory;
use Phalcon\Di;

$di     = new Di();
$config = new Ini('config.ini');

$di->set('config', $config);

$di->set(
    'db',
    function () {
        return Factory::load($this->config->database);
    }
);
```
上述代码返回数据库连接实例，这样做的好处是你可以在不修改应用中代码的情况下改变数据库连接甚至是数据库适配器。
## 查询记录(Finding Rows)
`Phalcon\Db`提供了多种查询方法。这种情况下，SQL必须遵循数据库引擎的特定语法：
```php
<?php

$sql = "SELECT id, name FROM robots ORDER BY name";

// 发送SQL语句到数据库
$result = $connection->query($sql);

// 打印robot的name字段
while ($robot = $result->fetch()) {
    echo $robot['name'];
}

// 获取结果集数组
$robots = $connection->fetchAll($sql);
foreach ($robots as $robot) {
    echo $robot['name'];
}

// 获取结果集中的第一条记录
$robot = $connection->fetchOne($sql);
```
默认情况下，调用这些方法会返回一个数组(关联+索引)。你可以调用`Phalcon\Db\Result::setFetchMode()`方法改变这种行为，该方法接收一个常量值，定义返回结果集的类型：
<table>
    <thead>
        <tr>
            <th>常量</th>
            <th>说明</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <code>Phalcon\\Db::FETCH_NUM</code>
            </td>
            <td>返回索引数组</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\\Db::FETCH_ASSOC</code>
            </td>
            <td>返回关联数组</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\\Db::FETCH_BOTH</code>
            </td>
            <td>返回数组(索引+关联)</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\\Db::FETCH_OBJ</code>
            </td>
            <td>返回对象</td>
        </tr>
    </tbody>
</table>

```php
<?php

$sql    = "SELECT id, name FROM robots ORDER BY name";
$result = $connection->query($sql);

$result->setFetchMode(Phalcon\Db::FETCH_NUM);
while ($robot = $result->fetch()) {
    echo $robot[0];
}
```
`Phalcon\Db::query()`方法返回一个`Phalcon\Db\Result\Pdo`实例。该对象封装了与返回结果集相关的所有功能，如遍历、查找特定行、统计总行数等。
```php
<?php

$sql    = "SELECT id, name FROM robots";
$result = $connection->query($sql);

// 遍历结果集
while ($robot = $result->fetch()) {
    echo $robot['name'];
}

// 查找第三行
$result->seek(2);
$robot = $result->fetch();

// 计算总行数
echo $result->numRows();
```
## 参数绑定(Binding Parameters)
`Phalcon\Db`支持参数绑定。尽管使用参数绑定会影响性能，但可以防止SQL注入。
支持字符串和占位符，参数绑定可以简单的实现如下：
```php
<?php

// 数字占位符
$sql    = "SELECT * FROM robots WHERE name = ? ORDER BY name";
$result = $connection->query(
    $sql,
    [
        'Wall-E',
    ]
);

// 指定占位符
$sql     = "INSERT INTO `robots`(name, year) VALUES(:name, :year)";
$success = $connection->query(
    $sql,
    [
        'name' => 'Astro Boy',
        'year' => 1952,
    ]
);
```
使用数字占位符时，你需要将它们定义为数字值(如1或2)，'1'或'2'会被视为字符串而非数字，导致占位符不能被成功替换。使用任何数据库适配器，数据都会被`Pdo::Quote()`自动转义。该方法会考虑到连接字符集，因此建议在连接选项或服务器配置中定义正确的字符集，错误的字符集会在存储或检索数据时产生不良影响。
此外，你也可以将参数直接传递给execute()/query()方法，这种情况下的绑定参数会直接传递给PDO：
```php
<?php

// PDO占位符
$sql    = "SELECT * FROM robots WHERE name = ? ORDER BY name";
$result = $connection->query(
    $sql,
    [
        1 => 'Wall-E',
    ]
);
```
## 特定类型占位符(Typed Placeholders)
占位符允许你执行参数绑定以避免SQL注入：
```php
<?php

$phql = "SELECT * FROM Store\Robots WHERE id > :id:";

$robots = $this->modelsManager->executeQuery(
    $phql,
    [
        'id' => 100,
    ]
);
```
某些数据库系统在使用占位符时需要执行额外操作，如指定绑定参数的类型：
```php
<?php

use Phalcon\Db\Column;

// ...

$phql   = "SELECT * FROM Store\Robots LIMIT :number:";
$robots = $this->modelsManager->executeQuery(
    $phql,
    [
        'number' => 10,
    ],
    Column::BIND_PARAM_INT
);
```
你可以在参数中使用类型化的占位符，而不用在`executeQuery()`方法中指定：
```php
<?php

$phql   = "SELECT * FROM Store\Robots LIMIT {number:int}";
$robots = $this->modelsManager->executeQuery(
    $phql,
    [
        'number' => 10,
    ]
);

$phql   = "SELECT * FROM Store\Robots WHERE name <> {name:str}";
$robots = $this->modelsManager->executeQuery(
    $phql,
    [
        'name' => $name,
    ]
);
```
如果你不需要指定绑定参数类型，你可以省略：
```php
<?php

$phql   = "SELECT * FROM Store\Robots WHERE name <> {name}";
$robots = $this->modelsManager->executeQuery(
    $phql,
    [
        'name' => $name,
    ]
);
```
类型化的占位符很强大，我们可以绑定静态数组，而无需将每个参数作为占位符单独传递：
```php
<?php

$phql   = "SELECT * FROM Store\Robots WHERE id IN ({ids:array})";
$robots = $this->modelsManager->executeQuery(
    $phql,
    [
        'ids' => [1, 2, 3, 4],
    ]
);
```
支持下列类型：
<table>
    <thead>
        <tr>
            <th>绑定类型</th>
            <th>绑定类型常量</th>
            <th>示例</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>str</td>
            <td>
                <code>Column::BIND_PARAM_STR</code>
            </td>
            <td>{name:str}</td>
        </tr>
        <tr>
            <td>int</td>
            <td>
                <code>Column::BIND_PARAM_INT</code>
            </td>
            <td>{number:int}</td>
        </tr>
        <tr>
            <td>double</td>
            <td>
                <code>Column::BIND_PARAM_DECIMAL</code>
            </td>
            <td>{price:double}</td>
        </tr>
        <tr>
            <td>bool</td>
            <td>
                <code>Column::BIND_PARAM_BOOL</code>
            </td>
            <td>{enabled:bool}</td>
        </tr>
        <tr>
            <td>blob</td>
            <td>
                <code>Column::BIND_PARAM_BLOB</code>
            </td>
            <td>{image:blob}</td>
        </tr>
        <tr>
            <td>null</td>
            <td>
                <code>Column::BIND_PARAM_NULL</code>
            </td>
            <td>{exists:null}</td>
        </tr>
        <tr>
            <td>array</td>
            <td>
                <code>Column::BIND_PARAM_STR数组</code>
            </td>
            <td>{codes:array}</td>
        </tr>
        <tr>
            <td>array-str</td>
            <td>
                <code>Column::BIND_PARAM_STR数组</code>
            </td>
            <td>{names:array}</td>
        </tr>
        <tr>
            <td>array-int</td>
            <td>
                <code>Column::BIND_PARAM_INT数组</code>
            </td>
            <td>{flags:array}</td>
        </tr>
    </tbody>
</table>

## 绑定参数类型转换(Catd bound parameters values)
默认情况下，绑定参数不会在PHP中转换为指定类型，
如，在`LIMIT/OFFSET`中给占位符传递一个字符串值就会导致错误：
```php
<?php

$number = '100';
$robots = $modelsManager->executeQuery(
    "SELECT * FROM Some\Robots LIMIT {number:int}",
    [
        'number' => $number,
    ]
);
```
这会导致异常：
```
Fatal error: Uncaught exception 'PDOException' with message 'SQLSTATE[42000]:
Syntax error or access violation: 1064 You have an error in your SQL syntax;
check the manual that corresponds to your MySQL server version for the right
syntax to use near ''100'' at line 1' in /Users/scott/demo.php:78
```
错误原因是'100'是一个字符串。可以先将值转换为整型：
```php
<?php

$number = '100';
$robots = $modelManager->executeQuery(
    "SELECT * FROM Some\Robots LIMIT {number:int}",
    [
        'number' => (int) $number,
    ]
);
```
要解决这个问题，需要开发者格外注意绑定参数类型及其如何传递。为了简化操作并避免异常，你可以指定`Phalcon`替你转换：
```php
<?php

\Phalcon\Db::setup(['forceCasting' => true]);
```
以下操作根据指定的绑定类型执行：
<table>
    <thead>
        <tr>
            <th>绑定类型</th>
            <th>操作</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <code>Column::BIND_PARAM_STR</code>
            </td>
            <td>将值转换为PHP字符串</td>
        </tr>
        <tr>
            <td>
                <code>Column::BIND_PARAM_INT</code>
            </td>
            <td>将值转换为PHP整型</td>
        </tr>
        <tr>
            <td>
                <code>Column::BIND_PARAM_BOOL</code>
            </td>
            <td>将值转换为PHP布尔值</td>
        </tr>
        <tr>
            <td>
                <code>Column::BIND_PARAM_DECIMAL</code>
            </td>
            <td>将值转换为PHP浮点数</td>
        </tr>
    </tbody>
</table>

从数据库返回的值在PDO中始终表示为字符串，无论该列值是数字还是布尔值。这种情况是因为某些列类型由于其大小限制而无法用PHP原来类型表示。例如，MySQL中的<code>BIGINT</code>可以存储无法用PHP 32位整型表示的大整数。所以，PDO和ORM默认将所有值作为字符串。你可以设置ORM自动将这些值转换为PHP实际类型：
```php
<?php

\Phalcon\Mvc\Model::setup(['castOnHydrate' => true]);
```
通过这种方式，你可以使用严格运算符或对变量类型进行假设：
```php
<?php

$robot = Robots::findFirst();
if (11 === $robot->id) {
    echo $robot->name;
}
```
## 插入/更新/删除记录(Inserting/Updading/Deleting Rows)
你可以使用原生SQL或类方法来插入、更新、删除记录：
```php
<?php

// 用原生SQL插入数据
$sql     = "INSERT INTO `robots`(`name`, `year`) VALUES('Astro Boy', 1952)";
$success = $connection->execute($sql);

// 使用占位符
$sql     = "INSERT INTO `robots`(`name`, `year`) VALUES(?, ?)";
$success = $connection->execute(
    $sql,
    [
        'Astro Boy',
        1952,
    ]
);

// 动态生成SQL语句
$success = $connection->insert(
    'robots',
    [
        'Astro Boy',
        1952,
    ],
    [
        'name',
        'year',
    ]
);

// 动态生成SQL语句(另一种语法)
$success = $connection->insertAsDict(
    'robots',
    [
        'name' => 'Astro Boy',
        'year' => 1952,
    ]
);

// 使用原生SQL更新数据
$sql     = "UPDATE `robots` SET `name` = 'Astro Boy' WHERE `id` = 101";
$success = $connection->execute($sql);

// 使用占位符
$sql     = "UPDATE `robots` SET `name` = ? WHERE `id` = ?";
$success = $connection->execute(
    $sql,
    [
        'Astro Boy',
        101,
    ]
);

// 动态生成SQL语句
$success = $connection->update(
    'robots',
    [
        'name',
    ],
    [
        'New Astro Boy',
    ],
    'id = 101' // 注意，这种情况下值不会被自动转义
);

// 条件中数据的转义
$success = $connection->update(
    'robots',
    [
        'name',
    ],
    [
        'New Astro Boy',
    ],
    [
        'conditions' => 'id = ?',
        'bind'       => [101],
        'bindTypes'  => [PDO::PARAM_INT], // 可选参数
    ]
);
$success = $connection->updateAsDict(
    'robots',
    [
        'name' => 'New Astro Boy',
    ],
    [
        'conditions' => 'id = ?',
        'bind'       => [101],
        'bindTypes'  => [PDO::PARAM_INT], // 可选参数
    ]
);

// 使用原生SQL删除记录
$sql     = "DELETE `robots` WHERE `id` = 101";
$success = $connection->execute($sql);

// 使用占位符
$sql     = "DELETE `robots` WHERE `id` = ?";
$success = $connection->execute($sql, [101]);

// 动态生成SQL语句
$success = $connection->delete(
    'robots',
    'id = ?',
    [
        101,
    ]
);
```
## 事务和嵌套事务(Transactions and Nested Transactions)
PDO支持事务处理，在事务内部执行数据库操作通常可以提高数据库的性能：
```php
<?php

try {
    // 开始事务
    $connection->begin();

    // 执行SQL语句
    $connection->execute("DELETE `robots` WHERE `id` = 101");
    $connection->execute("DELETE `robots` WHERE `id` = 102");
    $connection->execute("DELETE `robots` WHERE `id` = 103");

    // 如果一切顺利，提交事务
    $connection->commit();
} catch (Exception $e) {
    // 发生异常，回滚事务
    $connection->rollback();
}
```
除了标准事务，`Phalcon\Db`内置了嵌套事务(如果数据库支持)。当你再次调用`begin()`方法时，会创建一个嵌套事务：
```php
<?php

try {
    // 开始事务
    $connection->begin();

    // 执行SQL语句
    $connection->execute("DELETE `robots` WHERE `id` = 101");

    try {
        // 开始嵌套事务
        $connection->begin();

        // 嵌套事务中执行SQL语句
        $connection->execute("DELETE `robots` WHERE `id` = 102");
        $connection->execute("DELETE `robots` WHERE `id` = 103");

        // 创建保存点
        $connection->commit();
    } catch (Exception $e) {
        // 发生异常，回滚嵌套事务
        $connection->rollback();
    }

    // 继续执行更多SQL语句
    $connection->execute("DELETE `robots` WHERE `id` = 104");

    // 如果一切顺利，提交事务
    $connection->commit();
} catch (Exception $e) {
    // 发生异常，回滚事务
    $connection->rollback();
}
```
## 数据库事件(Database Events)
`Phalcon\Db`能够将事件发送给EventManager(如果存在)，某些事件返回false时，可能会终止操作。支持以下事件：
<table>
    <thead>
        <tr>
            <th>事件名称</th>
            <th>触发时机</th>
            <th>是否会终止操作</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <code>afterConnect</code>
            </td>
            <td>成功连接到数据库后</td>
            <td>否</td>
        </tr>
        <tr>
            <td>
                <code>beforeQuery</code>
            </td>
            <td>执行SQL语句前</td>
            <td>是</td>
        </tr>
        <tr>
            <td>
                <code>afterQuery</code>
            </td>
            <td>执行SQL语句后</td>
            <td>否</td>
        </tr>
        <tr>
            <td>
                <code>beforeDisconnect</code>
            </td>
            <td>关闭临时数据库连接前</td>
            <td>否</td>
        </tr>
        <tr>
            <td>
                <code>beginTransaction</code>
            </td>
            <td>事务开启前</td>
            <td>否</td>
        </tr>
        <tr>
            <td>
                <code>rollbackTransaction</code>
            </td>
            <td>事务回滚前</td>
            <td>否</td>
        </tr>
        <tr>
            <td>
                <code>commitTransaction</code>
            </td>
            <td>事务提交前</td>
            <td>否</td>
        </tr>
    </tbody>
</table>

将EventsManager绑定到数据库连接很简单，`Phalcon\Db`将触发`db`类型事件：
```php
<?php

use Phalcon\Db\Adapter\Pdo\Mysql as Connection;
use Phalcon\Events\Manager as EventsManager;

$eventsManager = new EventsManager();

// 监听所有数据库事件
$eventsManager->attch('db', $dbListener);

$connection = new Connection(
    [
        'host'     => 'localhost',
        'username' => 'root',
        'password' => 'secret',
        'dbname'   => 'invo',
    ]
);

// 将eventsManager分配给数据库适配器实例
$connection->setEventsManager($eventsManager);
```
数据库事件中，停止SQL操作非常有用。例如，你想在SQL执行前实现注入检查：
```php
<?php

use Phalcon\Events\Event;

$eventsManager->attch(
    'db:beforeQuery',
    function (Event $event, $connection) {
        $sql = $connection->getSQLStatement();

        // 检查SQL中是否有恶意关键字
        if (preg_match('/DROP|ALTER/i', $sql)) {
            // 不允许DROP/ALTERT操作
            return false;
        }

        return true;
    }
)
```
## 分析SQL语句(Profiling SQL Statements)
`Phalcon\Db`内置了性能分析组件`Phalcon\Db\Profiler`，用于分析数据库性能，以便诊断问题，发现瓶颈。使用`Phalcon\Db\Profiler`进行数据库分析相当容易：
```php
<?php

use Phalcon\Db\Profiler as DbProfiler;
use Phalcon\Events\Event;
use Phalcon\Events\Manager as EventsManager;

$eventsManager = new EventsManager();

$profiler = new DbProfiler();

// 监听所有数据库事件
$eventsManager->attch(
    'db',
    function (Event $event, $connection) use ($profiler) {
        if ($event->getType() === 'beforeQuery') {
            $sql = $connection->getSQLStatement();

            // 开始分析
            $profiler->startProfile($sql);
        }

        if ($event->getType() === 'afterQuery') {
            // 停止分析
            $profiler->stopProfile();
        }
    }
);

// 将事件管理器分配给数据库连接
$connection->setEventsManager($eventsManager);

$sql = "SELECT buyer_name, quantity, product_name FROM buyers LEFT JOIN products ON buyers.pid = products.id";

// 执行SQL语句
$connection->query($sql);

// 获取最后一个分析结果
$profile = $profiler->getLastProfile();

echo 'SQL Statement: ', $profile->getSQLStatement(), "\n";
echo 'Start Time: ', $profile->getInitialTime(), "\n";
echo 'Final Time: ', $profile->getFinalTime(), "\n";
echo 'Total Elapsed Time: ', $profile->getTotalElapsedSeconds(), "\n";
```
你还可以基于`Phalcon\Db\Profiler`创建自己的分析器，以实时统计发送到数据库的SQL语句：
```php
<?php

use Phalcon\Db\Profiler as Profiler;
use Phalcon\Db\Profiler\Item as Item;
use Phalcon\Evnets\Manager as EventsManager;

class DbProfiler extends Profiler
{
    // SQL语句发送给数据库服务器之前执行
    public function beforeStartProfile(Item $profile)
    {
        echo $profile->getSQLStatement();
    }

    // SQL语句发送到数据库服务器之后执行
    public function afterEndProfile(Item $profile)
    {
        echo $profile->getTotalElapsedSeconds();
    }
}

// 创建事件管理器
$eventsManager = new EventsManager();

// 创建事件监听器
$dbProfiler = new DbProfiler();

// 设置监听器监听所有数据库事件
$eventsManager->attch('db', $dbProfiler);
```
## 记录SQL语句(Logging SQL Statements)
使用诸如`Phalcon\Db`这样的高级抽象组件来访问数据库时，很难获知哪些语句被发送到了数据库。`Phalcon\Logger`配合`Phalcon\Db`使用，可以在数据库抽象层上提供日志记录功能。
```php
<?php

use Phalcon\Events\Event;
use Phalcon\Events\Manager as EventsManager;
use Phalcon\Logger;
use Phalcon\Logger\Adapter\File as FileLogger;

$eventsManager = new EventsManager();

$logger = new FileLogger('app/logs/db.log');

$eventsManager->attch(
    'db:beforeQuery',
    function (Event $event, $connection) use ($logger) {
        $sql = $connection->getSQLStatement();

        $logger->log($sql, Logger::INFO);
    }
);

// 将eventsManager分配给数据库适配器实例
$connection->setEventsManager($eventsManager);

// 执行SQL语句
$connection->insert(
    'products',
    [
        'Hot pepper',
        3.50,
    ],
    [
        'name',
        'price',
    ]
);
```
如上所述，文件`app/logs/db.log`将包含下列内容：
```
[Sun, 29 Apr 12 22:35:26 -0500][DEBUG][Resource Id #77] INSERT INTO products
(name, price) VALUES ('Hot pepper', 3.50)
```
## 自定义记录器(Implementing your own Logger)
你可以自定义记录器以记录数据库操作，通过创建一个实现了`log()`方法的类，该方法接受一个字符串作为第一个参数。你可以将记录器对象传递给`Phalcon\Db::setLogger()`，这样在执行任何SQL语句时将调用`log()`方法进行记录。
## 获取表/视图详情(Describing Tables/Views)
`Phalcon\Db`提供了获取表格、视图详情的方法：
```php
<?php

// 获取test_db库中的数据表
$tables = $connection->listTables('test_db');

// 表'robots'是否存在于当前库中
$exists = $connection->tableExists('robots');

// 获取'robots'表字段名称、类型、特性
$fields = $connection->describeColumns('robots');
foreach ($fields as $field) {
    echo 'Column Type: ', $field['Type'];
}

// 获取'robots'表索引
$indexes = $connection->describeIndexes('robots');
foreach ($indexes as $index) {
    print_r(
        $index->getColumns()
    );
}

// 获取'robots'表外键
$references = $connection->describeReferences('robots');
foreach ($references as $reference) {
    // 打印引用列
    print_r(
        $reference->getReferenceColumns()
    );
}
```
表详情和MySQL的describe命令返回的信息相似，包含如下信息：

<table>
    <thead>
        <tr>
            <th>Field</th>
            <th>Type</th>
            <th>Key</th>
            <th>Null</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>字段名称</td>
            <td>字段类型</td>
            <td>是否主键或索引列</td>
            <td>是否允许为空</td>
        </tr>
    </tbody>
</table>

对于被支持的数据库系统，也实现了获取视图详情的方法：
```php
<?php

// 获取test_db库中的视图
$tables = $connection->listViews('test_db');

// 视图'robots'是否存在于当前库中
$exists = $connection->viewExists('robots');
```
## 创建、修改、删除表(Creating/Alerting/Dropping Tables)
不同的数据库系统(MySQL，Postgresql等)通过CREATE、ALTER、DROP命令提供了用于创建、修改、删除数据表的功能。SQL语法因数据库而异。`Phalcon\Db`为编辑表提供了统一接口，无需区分不同数据库系统的SQL语法。
### 创建表(Creating Tables)
下面例子展示如何创建表：
```php
<?php

use Phalcon\Db\Column as Column;

$connection->createTable(
    'robots',
    null,
    [
        'columns' => [
            new Column(
                'id',
                [
                    'type'          => Column::TYPE_INTEGER,
                    'size'          => 10,
                    'notNull'       => true,
                    'autoIncrement' => true,
                    'primary'       => true,
                ]
            ),
            new Column(
                'name',
                [
                    'type'    => Column::TYPE_VARCHAR,
                    'size'    => 70,
                    'notNull' => true,
                ]
            ),
            new Column(
                'year',
                [
                    'type'    => Column::TYPE_INTEGER,
                    'size'    => 11,
                    'notNull' => true,
                ]
            ),
        ],
    ]
);
```
`Phalcon\Db::createTable()`接收一个描述数据表的关联数组，用`Phalcon\Db\Column`类创建字段，下表列出了定义字段的选项：

<table>
    <thead>
        <tr>
            <th>选项</th>
            <th>说明</th>
            <th>是否可选</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <code>type</code>
            </td>
            <td>字段类型，必须是<code>Phalcon\Db\Column</code>常量</td>
            <td>否</td>
        </tr>
        <tr>
            <td>
                <code>primary</code>
            </td>
            <td>是否主键</td>
            <td>是</td>
        </tr>
        <tr>
            <td>
                <code>size</code>
            </td>
            <td><code>VARCHAR</code>或<code>INTEGER</code>类型的字段定义字段长度</td>
            <td>是</td>
        </tr>
        <tr>
            <td>
                <code>scale</code>
            </td>
            <td><code>DEMICAL</code>或<code>NUMBER</code>类型字段定义数据精度</td>
            <td>是</td>
        </tr>
        <tr>
            <td>
                <code>unsigned</code>
            </td>
            <td><code>INTEGER</code>类型字段定义是否有符号，该选项不适用于其他类型字段</td>
            <td>是</td>
        </tr>
        <tr>
            <td>
                <code>notNull</code>
            </td>
            <td>字段是否非空</td>
            <td>是</td>
        </tr>
        <tr>
            <td>
                <code>default</code>
            </td>
            <td>默认值</td>
            <td>是</td>
        </tr>
        <tr>
            <td>
                <code>autoIncrement</code>
            </td>
            <td>是否自增</td>
            <td>是</td>
        </tr>
        <tr>
            <td>
                <code>bind</code>
            </td>
            <td><code>BIND_TYPE_*</code>常量定义字段在保存前如何绑定数据</td>
            <td>是</td>
        </tr>
        <tr>
            <td>
                <code>first</code>
            </td>
            <td>把字段设置为表的第一个字段</td>
            <td>是</td>
        </tr>
        <tr>
            <td>
                <code>after</code>
            </td>
            <td>设置字段放在指定字段之后</td>
            <td>是</td>
        </tr>
    </tbody>
</table>

`Phalcon\Db`支持下列字段类型：

- `Phalcon\Db\Column::TYPE_INTEGER`
- `Phalcon\Db\Column::TYPE_DATE`
- `Phalcon\Db\Column::TYPE_VARCHAR`
- `Phalcon\Db\Column::TYPE_DECIMAL`
- `Phalcon\Db\Column::TYPE_DATETIME`
- `Phalcon\Db\Column::TYPE_CHAR`
- `Phalcon\Db\Column::TYPE_TEXT`

传入`Phalcon\Db::createTable()`方法的关联数组可能包含下列索引：

<table>
    <thead>
        <tr>
            <th>索引</th>
            <th>说明</th>
            <th>是否可选</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <code>columns</code>
            </td>
            <td>由<code>Phalcon\\Db\\Column</code>定义的字段组成数组</td>
            <td>否</td>
        </tr>
        <tr>
            <td>
                <code>indexes</code>
            </td>
            <td>由<code>Phalcon\\Db\\Index</code>定义的表索引组成的数组</td>
            <td>是</td>
        </tr>
        <tr>
            <td>
                <code>references</code>
            </td>
            <td>由<code>Phalcon\\Db\\Reference</code>定义的表引用(外键)组成的数组</td>
            <td>是</td>
        </tr>
        <tr>
            <td>
                <code>options</code>
            </td>
            <td>包含表创建选项的数组，这些选项通常与数据库迁移相关</td>
            <td>是</td>
        </tr>
    </tbody>
</table>

### 编辑表(Alerting Tables)
随着应用程序越来越庞杂，你可能需要调整数据库，作为重构或添加新功能的一部分。并非所有数据库系统都允许修改列或者新增列，`Phalcon\Db`也受到这些限制：
```php
<?php

use Phalcon\Db\Column as Column;

// 新增列
$connection->addColumn(
    'robots',
    null,
    new Column(
        'robot_type',
        [
            'type'    => Column::TYPE_VARCHAR,
            'size'    => 32,
            'notNull' => true,
            'after'   => 'name',
        ]
    )
);

// 编辑列
$connection->modifyColumn(
    'robots',
    null,
    new Column(
        'name',
        [
            'type'    => Column::TYPE_VARCHAR,
            'size'    => 40,
            'notNull' => true,
        ]
    )
);

// 删除'name'列
$connection->dropColumn(
    'robots',
    null,
    'name'
);
```
### 删除表(Dropping Tables)
删除表示例：
```php
<?php

// 从当前库中删除'robots'表
$connection->dropTable('robots');

// 从'machines'库中删除'robots'表
$connection->dropTable('robots', 'machines');
```