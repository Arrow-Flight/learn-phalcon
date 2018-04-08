# 使用模型
模型表示应用程序信息(数据)和这些数据的处理规则，主要用于管理与对应数据表的交互规则。多数情况下，数据库中的每张表都有一个模型与之对应。应用程序中的大部分业务逻辑集中在模型中。

Phalcon应用中，`Phalcon\Mvc\Model`是所有模型的基类。它提供了数据库独立、基础CRUD、高级查找、模型关联以及其他服务。

`Phalcon\Mvc\Model`将方法动态的转换为对应的数据库操作，规避了使用SQL语句的必要。

模型使用数据库高级抽象层，如果你想要使用更为底层的方式操作数据库，请参考`Phalcon\Db`组件文档。
## 创建模型(Creating Models)
模型继承`Phalcon\Mvc\Model`类，以大驼峰格式命名。
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class RobotParts extends Model
{

}
```
如果你在使用PHP 5.4、5.5版本，建议在模型中声明对应数据表的所有字段，以节约内存。

模型`Store\Toys\RobotParts`默认映射`robot_parts`表，可以调用`setSource()`方法手动指定映射表：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class RobotParts extends Model
{
    public function initialize()
    {
        $this->setSource('toys_robot_parts');
    }
}
```
模型`RobotParts`现在映射`toys_robot_parts`表。`initialize()`方法有助于在模型中创建自定义行为，如为模型指定映射表。

`initialize()`方法在请求期间只调用一次，目的是为该模型的所有实例执行初始化操作。如果每次实例化模型的时候都需要进行初始化，可以使用`onConstruct()`方法：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class RobotParts extends Model
{
    public function onConstruct()
    {
        // ...
    }
}
```
### 公共属性和Setters、Getters方法(Public properties vs. Setters/Getters)
模型可以定义公共属性，意味着在任何实例化了模型的代码中都可以读写模型的公共属性：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public $id;

    public $name;

    public $price;
}
```
另一种实现方式是`getters`和`setters`方法，控制哪些模型属性可以公开访问。这种方式的好处是，开发者可以在对模型属性进行写操作时执行转换和验证，这是使用公共属性方式无法实现的。此外，`getters`和`setters`可以在不改动模型和接口的前提下，应对未来可能的改动。如果字段名称改变，唯一需要的改动的地方是`getters`和`setters`中引用的模型私有属性。
```php
<?php

namespace Store\Toys;

use InvalidArgumentException;
use Phalcon\Mvc\Model;

class Robots extends Model
{
    protected $id;

    protected $name;

    protected $price;

    public function getId()
    {
        return $this->id;
    }

    public function setName($name)
    {
        // name不能太短
        if (strlen($name) < 10) {
            throw new InvalidArgumentException(
                'The name is too short'
            );
        }

        $this->name = $name;
    }

    public function getName()
    {
        return $this->name;
    }

    public function setPrice($price)
    {
        // price不能为负
        if ($price < 0) {
            throw new InvalidArgumentException(
                "Price can't be negative"
            );
        }

        $this->price = $price;
    }

    public function getPrice()
    {
        // 返回前，将该值转换为double类型
        return (double) $this->price;
    }
}
```
虽然公共属性在开发中的复杂度更低，但是`getters`和`setters`可以大大提高程序的测试性、扩展性和可维护性。开发者可以根据需求决定哪种方式更适合他们的应用。ORM兼容这两种方式。

使用`getters`和`setters`时，属性名中的下划线可能会导致问题。

如果你在属性名中使用下划线，在声明`getters`和`setters`魔术方法时，仍然要使用驼峰格式(`$model->getPropertyName()`代替`$model->getProperty_name()`，`$model->findByPropertyName()`代替`$model->findByProperty_name()`等)。大多数系统推荐驼峰写法，而不是下划线写法，所以建议你按照文档中的写法来命名属性。你可以使用字段映射(如上所述)以确保属性正确映射到数据表中对应字段。

## 理解记录对象(Understanding Records To Objects)
模型的每一个实例代表数据表中的一条记录，你可以通过读取模型对象属性来访问记录数据。例如，表`robots`有如下记录：
```sql
mysql> select * from robots;
+----+------------+------------+------+
| id | name       | type       | year |
+----+------------+------------+------+
|  1 | Robotina   | mechanical | 1972 |
|  2 | Astro Boy  | mechanical | 1952 |
|  3 | Terminator | cyborg     | 2029 |
+----+------------+------------+------+
3 rows in set (0.00 sec)
```
你可以通过主键查找某条记录：
```php
<?php

use Store\Toys\Robots;

// 查找id = 3的记录
$robot = Robots::findFirst(3);

// 输出'Terminator'
echo $robot->name;
```
一旦记录存储在内存中，你可以修改其中的数据并保存：
```php
<?php

use Store\Toys\Robots;

$robot = Robots::findFirst(3);

$robot->name = 'RoboCop';

$robot->save();
```
如你所见，`Phalcon\Mvc\Model`为web应用提供了数据库高级抽象层，不需要使用原生SQL语句。
## 查找记录(Finding Records)
`Phalcon\Mvc\Model`提供了多种查询记录的方法，下面例子演示如何用模型查找一条或多条记录：
```php
<?php

use Store\Toys\Robots;

// 查询所有记录
$robots = Robots::find();
echo 'There are ', count($robots), "\n";

// 查询type = 'mechanical'的记录
$robots = Robots::find("type = 'mechanical'");
echo 'There are ', count($robots), "\n";

// 获取type = 'virtual'的记录，根据name排序
$robots = Robots::find(
    [
        "type = 'virtual'",
        'order' => 'name',
    ]
);
foreach ($robots as $robot) {
    echo $robot->name, "\n";
}

// 获取type = 'virtual'的前100条记录，根据name排序
$robots = Robots::find(
    [
        "type = 'virtual'",
        'order' => 'name',
        'limit' => 100,
    ]
);
foreach ($robots as $robot) {
    echo $robot->name, "\n";
}
```
如果你想通过外部数据(如用户输入)或变量查找记录，必须使用数据绑定。

你可以使用`findFirst()`方法，获取满足给定条件的第一条记录：
```php
<?php

use Store\Toys\Robots;

// 获取robots表的第一条记录
$robot = Robots::findFirst();
echo 'The robot name is ', $robot->name, "\n";

// 获取robots表中type = 'mechanical'的第一条记录
$robot = Robots::findFirst("type = 'mechanical'");
echo 'The first mechanical robot name is ', $robot->name, "\n";

// 获取robots表中type = 'virtual'的第一条记录，根据name排序
$robot = Robots::findFirst(
    [
        "type = 'virtual'",
        'order' => 'name',
    ]
);

echo 'The first virtual robot name is ', $robot->name, "\n";
```
`find()`方法和`findFirst()`方法都接受一个包含指定搜索条件的关联数组：
```php
<?php

use Store\Toys\Robots;

$robots = Robots::findFirst(
    [
        "type = 'virtual'",
        'order' => 'name DESC',
        'limit' => 30,
    ]
);

$robots = Robots::find(
    [
        'conditions' => 'type = ?1',
        'bind'       => [
            1 => 'virtual',
        ],
    ]
);
```
有下列查询选项：

<table>
    <thead>
        <tr>
            <th>参数</th>
            <th>说明</th>
            <th>示例</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <code>
                    conditions
                </code>
            </td>
            <td>
                查询操作的搜索条件，用于提取符合指定条件的记录。默认情况下，<code>Phalcon\Mvc\Model</code>假定第一个参数就是搜索条件
            </td>
            <td>
                <code>
                    'conditions' => "name LIKE 'steve%'"
                </code>
            </td>
        </tr>
        <tr>
            <td>
                <code>
                    columns
                </code>
            </td>
            <td>
                获取模型中的指定字段，而不是所有字段。使用此选项时，会返回一个不完整对象。
            </td>
            <td>
                <code>
                    'columns' => 'id, name'
                </code>
            </td>
        </tr>
        <tr>
            <td>
                <code>
                    bind
                </code>
            </td>
            <td>
               参数绑定与<code>conditions</code>一起使用，通过替换占位符、转义特殊字符从而提高安全性
            </td>
            <td>
                <code>
                    'bind' => ['status' => 'A', 'type' => 'some-time']
                </code>
            </td>
        </tr>
        <tr>
            <td>
                <code>
                    bindTypes
                </code>
            </td>
            <td>
                使用参数绑定时，可以使用这个参数为绑定参数定义额外的类型限制，从而提高安全性
            </td>
            <td>
                <code>
                    'bindTypes' => [Column::BIND_PARAM_STR, Column::BIND_PARAM_INT]
                </code>
            </td>
        </tr>
        <tr>
            <td>
                <code>
                    order
                </code>
            </td>
            <td>
                用于对结果集进行排序，使用逗号分隔多个字段
            </td>
            <td>
                <code>
                    'order' => 'name DESC, status'
                </code>
            </td>
        </tr>
        <tr>
            <td>
                <code>
                    limit
                </code>
            </td>
            <td>
                将查询结果的数量限制在一定范围内
            </td>
            <td>
                <code>
                    'limit' => 10
                </code>
            </td>
        </tr>
        <tr>
            <td>
                <code>
                    offset
                </code>
            </td>
            <td>
                设定查询结果偏移量
            </td>
            <td>
                <code>
                    'offset' => 5
                </code>
            </td>
        </tr>
        <tr>
            <td>
                <code>
                    group
                </code>
            </td>
            <td>
                允许跨记录收集数据，并将结果集按一个或多个字段分组
            </td>
            <td>
                <code>
                    'group' => 'name, status'
                </code>
            </td>
        </tr>
        <tr>
            <td>
                <code>
                    for_update
                </code>
            </td>
            <td>
                使用此选项，<code>Phalcon\Mvc\Model</code>将读取最新的可用数据，并为读取到的每一条记录设置独占锁
            </td>
            <td>
                <code>
                    'for_update' => true
                </code>
            </td>
        </tr>
        <tr>
            <td>
                <code>
                    shared_lock
                </code>
            </td>
            <td>
                使用此选项，<code>Phalcon\Mvc\Model</code>将读取最新的可用数据，并为读取到的每一条记录设置共享锁
            </td>
            <td>
                <code>
                    'shared_lock' => true
                </code>
            </td>
        </tr>
        <tr>
            <td>
                <code>
                    cache
                </code>
            </td>
            <td>
                缓存结果集，减少对数据库的持续访问
            </td>
            <td>
                <code>
                    'cache' => ['lifetime' => 3600, 'key' => 'my-find-key']
                </code>
            </td>
        </tr>
        <tr>
            <td>
                <code>
                    hydration
                </code>
            </td>
            <td>
                设置结果集返回模式
            </td>
            <td>
                <code>
                    'hydration' => Resultset::HYDRATE_OBJECTS
                </code>
            </td>
        </tr>
    </tbody>
</table>

如果你愿意，除了使用参数数组，还可以使用面向对象的方式创建查询：
```php
<?php

use Store\Toys\Robots;

$robots = Robots::query()
    ->where('type = :type:')
    ->addWhere('year < 2000')
    ->bind(['type' => 'machanical'])
    ->order('name')
    ->execute();
```
静态方法`query()`返回一个IDE自动完成友好的`Phalcon\Mvc\Model\Criteria`对象。
所有查询在内部都以PHQL查询的方式处理。PHQL是一种高级的、面向对象的类SQL语言，这种语言提供了许多功能来执行查询，如join其他模型，定义分组，添加聚合等。
最后，还有一个`findFirstBy<property-name>()`方法，该方法扩展了`findFirst()`方法，它允许你通过使用方法中的属性名称并向它传递一个包含要在该字段中搜索数据的参数，从表中快速执行检索。以上面的Robots模型为例：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public $id;

    public $name;

    public $price;
}
```
这里有三个属性：`$id`，`$name`和`$price`，假设你想要检索name为'Terminator'的第一条记录，代码如下：
```php
<?php

use Store\Toys\Robots;

$name = 'Terminator';

$robot = Robots::findFirstByName($name);

if ($robot) {
    echo 'The first robot with the name ', $name . ' cost ' . $robot->price, '.';
} else {
    echo 'There were no robots found in our table with the name ' . $name . '.';
}
```
请注意，我们在调用的方法中使用了Name并传递了变量`$name`，表中name字段值为`$name`的记录就是我们要查找的。
### 模型结果集(Model Resultsets)
`findFirst()`方法返回被调用类的实例(如果有结果返回)，而`find()`方法返回`Phalcon\Mvc\Model\Resultsets\Simple`对象，该对象封装了结果集应有的所有功能，如遍历，找特定记录，统计等。

这些对象比一般数组功能强大，`Phalcon\Mvc\Model\Resultset`一个最大的特点是，任何时刻，只有一条记录保存在内存中。这对内存管理有极大帮助，特别是在处理大批量数据时。
```php
<?php

use Store\Toys\Robots;

// 获取所有记录
$robots = Robots::find();

// foreach遍历
foreach ($robots as $robot) {
    echo $robot->name, "\n";
}

// while遍历
while ($robots->valid()) {
    $robot = $robots->current();

    echo $robot->name, "\n";

    $robots->next();
}

// 结果集计数
echo count($robots);

// 另一种结果集计数方法
echo $robots->count();

// 移动游标到第三条记录
$robots->seek(2);

$robot = $robots->current();

// 通过位置访问结果集中的记录
$robot = $robots[5];

// 检查指定位置是否有记录
if (isset($robots[3])) {
    $robot = $robots[3];
}

// 获取结果集中第一条记录
$robot = $robots->getFirst();

// 获取结果集中最后一条记录
$robot = $robots->getLast();
```
Phalcon结果集模拟游标，你可以通过访问其位置或内部指针获取任一条记录。注意，某些数据库系统不支持游标，这会导致查询被反复执行，以重置游标到初始位置并获取被请求位置的记录。同样的，遍历多少次结果集，查询便要执行多少次。

将大量查询结果保存在内存中会消耗太多资源，因此，在某些情况下，以32条记录为一块从数据库中获取记录，可以降低重复执行请求的内存消耗。

注意，结果集可以序列化后存储在缓存中，`Phalcon\Cache`有助于实现该需求。但是序列化数据会导致`Phalcon\Mvc\Model`会以数组形式保存从数据库中检索到的数据，这会导致更多的内存消耗。
```php
<?php

// 查询表parts所有记录
$parts = Parts::find();

// 将结果集保存到文件中
fild_put_contents(
    'cache.txt',
    serialize($parts)
);

// 从文件中获取结果集
$parts = unserialize(
    file_get_contents('cache.txt')
);

// 遍历
foreach ($parts as $part) {
    echo $part->id;
}
```
### 自定义结果集(Custom Resultsets)
有时候应用程序需要对数据库中检索到的数据进行额外处理。以前，我们只需要扩展模型或将功能封装在模型或trait当中，然后返回一组经过转换的数据。

通过自定义结果集，你不需要再如此处理。自定义结果集将封装原本封装在模型中并能被其他模型重用的功能，这样可以简化代码。如此，`find()`方法返回自定义对象，而不再返回`Phalcon\Mvc\Model\Resultset`对象。Phalcon允许你在模型中定义`getResultsetClass()`方法来实现此操作。

首先，声明resultset类：
```php
<?php

namespace Application\Mvc\Model\Resultset;

use Phalcon\Mvc\Model\Resultset\Simple;

class Custom extends Simple
{
    public function getSomeData()
    {
        /** CODE */
    }
}
```
模型中，在`getResultsetClass()`方法里设置resultset类，如下：
```php
<?php

namespace Phalcon\Test\Models\Statistics;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function getSource()
    {
        return 'robots';
    }

    public function getResultsetClass()
    {
        return 'Application\Mvc\Model\Resultset\Custom';
    }
}
```
最后，你的代码里应包含如下内容：
```php
<?php

/**
 * 查找robots表记录
 */
$robots = Robots::find(
    [
        'conditions' => 'date between "2017-01-01" AND "2017-12-31"',
        'order'      => 'date',
    ]
);

/**
 * 传递数据到视图
 */
$this->view->mydata = $robots->getSomeData();
```
### 过滤结果集(Filtering Resultsets)
过滤数据最有效的方法之一是设置搜索条件，数据库将使用表索引更快的返回数据。Phalcon允许你使用PHP以及数据库不支持的方式过滤数据:
```php
<?php

$customers = Customers::find();

$customers = $customers->filter(
    function ($customer) {
        // 只返回e-mail合法的记录
        if (filter_var($customer->email, FILTER_VALIDATE_EMAIL)) {
            return $customer;
        }
    }
);
```
### 参数绑定(Binding Parameters)
`Phalcon\Mvc\Model`支持参数绑定，建议使用这种方法以避免SQL注入，字符串占位符和数字占位符均被支持。参数绑定简单实现如下：
```php
<?php

use Store\Toys\Robots;

// 字符串占位符
$robots = Robots::find(
    [
        'name = :name: AND type = :type:',
        'bind' => [
            'name' => 'Robotina',
            'type' => 'maid',
        ],
    ]
);

// 数字占位符
$robots = Robots::find(
    [
        'name = ?1 AND type = ?2',
        'bind' => [
            1 => 'Robotina',
            2 => 'maid',
        ],
    ]
);

// 同时使用字符串占位符和数字占位符
$robots = Robots::find(
    [
        'name = :name: AND type = ?1',
        'bind' => [
            'name' => 'Robotina',
            1      => 'maid',
        ],
    ]
);
```
使用数字占位符时，你需要将它们定义成整数形式，即1或2。而'1'或'2'会被当成字符串，所以占位符不能被成功替换。

字符串会自动使用PDO转义，该功能会考虑连接字符集，因此建议在连接参数或数据库配置中定义正确字符集，因为错误字符集会在存储和检索数据时产生不良影响。

你还可以设置`bindTypes`参数，这允许你定义参数如何根据其类型绑定:
```php
<?php

use Phalcon\Db\Column;
use Store\Toys\Robots;

// 绑定参数
$parameters = [
    'name' => 'Robotina',
    'year' => 2008,
];

// 参数类型转换
$types = [
    'name' => Column::BIND_PARAM_STR,
    'year' => Column::BIND_PARAM_INT,
];

// 字符串占位符
$robots = Robots::find(
    [
        'name = :name: AND year = :year:',
        'bind'      => $parameters,
        'bindTypes' => $types,
    ]
);
```
由于默认的绑定类型是`Phalcon\Db\Column::BIND_PARAM_STR`，如果所有字段都是字符串类型，则没有必要指定`bindTypes`参数。

如果使用数组作为绑定参数，则数组必须是键名从0开始的索引数组:
```php
<?php

use Store\Toys\Robots;

$array = ['a', 'b', 'c']; // $array: [[0] => 'a', [1] => 'b', [2] => 'c']

unset($array[1]); // $array: [[0] => 'a', [2] => 'c']

// 现在必须为重建数组索引
$array = array_values($array); // $array: [[0] => 'a', [1] => 'c']

$robots = Robots::find(
    [
        'letter IN ({letter:array})',
        'bind' => [
            'letter' => $array,
        ],
    ]
);
```
参数绑定除了可用于所有查询方法，如`find()`和`findFirst()`外，还可用于`count()`，`sum()`，`average()`等计算方法。

使用`finders`时，会自动使用参数绑定：
```php
<?php

use Store\Toys\Robots;

// 显式使用参数绑定
$robots = Robots::find(
    [
        'name = ?0',
        'bind' => [
            'Ultron',
        ],
    ]
);

// 隐式使用参数绑定
$robots = Robots::findByName('Ultron');
```
## 初始化已获取记录(Initializing / Preparing fetched records)
有时从数据库获取记录之后，在数据被应用程序使用之前，需要对数据进行初始化。可以在模型中实现`afterFetch()`方法，实例化模型时会执行该方法，并将数据传递给它：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public $id;

    public $name;

    public $status;

    public function beforeSave()
    {
        // 将数组转换成字符串
        $this->status = join(',', $this->status);
    }

    public function afterFetch()
    {
        // 将字符串转换成数组
        $this->status = explode(',', $this->status);
    }

    public function afterSave()
    {
        // 将字符串转换成数组
        $this->status = explode(',', $this->status);
    }
}
```
如果你使用`getters/setters`代替公共属性，或同时使用它们，你可以在字段被访问时初始化字段：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public $id;

    public $name;

    public $status;

    public function getStatus()
    {
        return explode(',', $this->status);
    }
}
```
## 生成运算(Generating Calculations)
运算(或聚合)是数据库中常用的辅助方法，如`COUNT`，`SUM`，`MAX`，`MIN`和`AVG`。`Phalcon\Mvc\Model`可以直接使用这些方法。
Count示例：
```php
<?php

// 表employees总记录数
$rowcount = Employees::count();

// 表employees共有多少不同areas值
$rowcount = Employees::count(
    [
        'distinct' => 'area',
    ]
);

// 表employees共有多少area为'Testing'的记录
$rowcount = Employees::count(
    'area = "Testing"'
);

// 按area分组统计表employees记录
$group = Employees::count(
    [
        'group' => 'area',
    ]
);
foreach ($group as $row) {
    echo 'There are ', $row->rowcount, ' in ', $row->area;
}

// 按area分组统计表employees记录，并根据数目排序
$group = Employees::count(
    [
        'group' => 'area',
        'order' => 'rowcount',
    ]
);

// 使用参数绑定避免SQL注入
$group = Employees::count(
    [
        'type > ?0',
        'bind' => [
            $type,
        ],
    ]
);
```
Sum示例：
```php
<?php

// 所有employees的salaries总和
$total = Employees::sum(
    [
        'column' => 'salary',
    ]
);

// area = 'Sales'的所有employees的salaries总和
$total = Employees::sum(
    [
        'column'     => 'salary',
        'conditions' => 'area = "Sales"',
    ]
);

// 根据area分组统计salaries
$group = Employees::sum(
    [
        'column' => 'salary',
        'group'  => 'area',
    ]
);
foreach ($group as $row) {
    echo 'The sum of salaries of the ', $row->area, ' is ', $row->sumatory;
}

// 根据area分组统计salaries，salaries由高到低排序
$group = Employees::sum(
    [
        'column' => 'salary',
        'group'  => 'area',
        'order'  => 'sumatory DESC',
    ]
);

// 使用参数绑定避免参数绑定
$group = Employees::sum(
    [
        'conditions' => 'area > ?0',
        'bind'       => [
            $area,
        ],
    ]
);
```
Average示例：
```php
<?php

// 所有employees的平均salary
$average = Employees::average(
    [
        'column' => 'salary',
    ]
);

// area = 'Sales'的employees的平均salary
$average = Employees::average(
    [
        'column'     => 'salary',
        'conditions' => 'area = "Sales"',
    ]
);

// 使用参数绑定避免SQL注
$average = Employees::average(
    [
        'column'     => 'age',
        'conditions' => 'area > ?0',
        'bind'       => [
            $area,
        ],
    ]
);
```
Max / Min示例：
```php
<?php

// 所有employees中age最大的
$age = Employees::maximum(
    [
        'column' => 'age',
    ]
);

// area = 'Sales'的employees中age最大的
$age = Employees::maximum(
    [
        'column'     => 'age',
        'conditions' => 'area = "Sales"',
    ]
);

// 所有employees中salary最低的
$salary = Employees::minimum(
    [
        'column' => 'salary',
    ]
);
```
## 创建 / 更新记录(Creating / Updating Records)
`Phalcon\Mvc\Model::save()`方法会根据记录是否存在于模型映射表中而创建 / 更新记录，`Phalcon\Mvc\Model`的创建和更新方法会在内部调用该方法。为此，必须在实体中正确定义主键，以确定是否应创建记录还是更新记录。

该方法会执行相关验证器，虚拟外键和模型中定义的事件：
```php
<?php

use Store\Toys\Robots;

$robot = new Robots();

$robot->type = 'mechanical';
$robot->name = 'Astro Boy';
$robot->year = 1952;

if ($robot->save() === false) {
    echo "Umh, We can't store robots right now: \n";

    $messages = $robot->getMessages();

    foreach ($messages as $message) {
        echo $message, "\n";
    }
} else {
    echo 'Great, a new robot was saved successfully!';
}
```
直接传递或者通过属性数组传递的值会根据其数据类型自动被转义 / 过滤，所以你可以传递一个不安全的数组而不用担心SQL注入：
```php
<?php

use Store\Toys\Robots;

$robot = new Robots();

$robot->save($_POST);
```
毫无防护的批量值传递可能会允许攻击者设置任何数据库字段的值，仅在你允许用户插入 / 更新模型中所有字段的情况下使用上述功能，即使这些字段不是使用表单提交的。

可以在`save()`方法中设置额外参数，以设置批量值传递时，执行插入 / 更新操作的白名单字段。
```php
<?php

use Store\Toys\Robots;

$robot = new Robots();

$robot->save(
    $_POST,
    [
        'name',
        'type',
    ]
);
```
### 创建 / 更新执行结果(Create / Update with Confidence)
应用程序高并发时，创建记录操作可能会变成更新操作。使用`Phalcon\Mvc\Model::save()`方法保存记录时，就可能发生这种情况。如果想确保执行创建或更新，可以使用`create()`和`update()`方法替换`save()`：
```php
<?php

use Store\Toys\Robots;

$robot = new Robots();

$robot->type = 'mechanical';
$robot->name = 'Astro Boy';
$robot->year = 1952;

// 仅创建记录
if ($robot->create() === false) {
    echo "Umh, We can't store robots right now: \n";

    $messages = $robot->getMessages();

    foreach ($messages as $message) {
        echo $message, "\n";
    }
} else {
    echo 'Great, a new robot was created successfully!';
}
```
`create()`方法和`update()`方法同样接受一个数组作为参数。
## 删除记录(Deleting Records)
`Phalcon\Mvc\Model::delete()`方法允许删除记录，使用示例：
```php
<?php

use Store\Toys\Robots;

$robot = Robots::findFirst(11);

if ($robot !== false) {
    if ($robot->delete() === false) {
        echo "Sorry, we can't delete the robot right now: \n";

        $messages = $robot->getMessages();

        foreach ($messages as $message) {
            echo $message, "\n";
        }
    } else {
        echo 'The robots was deleted successfully!';
    }
}
```
也可以通过使用foreach遍历结果集来删除多条记录：
```php
<?php

use Store\Toys\Robots;

$robots = Robots::find(
    "type = 'mechanical'"
);

foreach ($robots as $robot) {
    if ($robot->delete() === false) {
        echo "Sorry, we can't delete the robot right now: \n";

        $messages = $robot->getMessages();

        foreach ($messages as $message) {
            echo $message, "\n";
        }
    } else {
        echo 'The robot was deleted successfully!';
    }
}
```
以下事件可以用于定义在执行删除操作时，可以执行的自定义业务规则：

<table>
    <thead>
        <tr>
            <th>操作</th>
            <th>事件名称</th>
            <th>能否终止操作</th>
            <th>说明</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                删除
            </td>
            <td>
                afterDelete
            </td>
            <td>
                否
            </td>
            <td>
                删除操作后执行
            </td>
        </tr>
        <tr>
            <td>
                删除
            </td>
            <td>
                beforeDelete
            </td>
            <td>
                是
            </td>
            <td>
                删除操作前执行
            </td>
        </tr>
    </tbody>
</table>

通过上述事件，可以在模型中定义业务规则：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function beforeDelete()
    {
        if ($this->status === 'A') {
            echo "The robot is active, it can't be deleted";

            return false;
        }

        return true;
    }
}
```
## Hydrations模式(Hydration Modes)
如前所述，结果集是完整对象的集合，这意味着每个返回结果都是一个对象，代表数据表中的一行。这些对象可以修改并永久保存：
```php
<?php

use Store\Toys\Robots;

$robots = Robots::find();

// 操作完整对象结果集
foreach ($robots as $robot) {
    $robot->year = 2000;

    $robot->save();
}
```
有时记录只能以只读模式呈现给用户，这种情况下，改变记录的展现方式可能有助于用户处理数据。用于表示结果集中返回的对象的策略成为'hydration mode'：
```php
<?php

use Phalcon\Mvc\Model\Resultset;
use Store\Toys\Robots;

$robots = Robots::find();

// 返回数组
$robots->setHydrateMode(
    Resultset::HYDRATE_ARRAYS
);

foreach ($robots as $robot) {
    echo $robot['year'], PHP_EOL;
}

// 返回stdClass对象
$robots->setHydrateMode(
    Resultset::HYDRATE_OBJECTS
);

foreach ($robots as $robot) {
    echo $robot->year, PHP_EOL;
}

// 返回模型实例
$robots->setHydrateMode(
    Resultset::HYDRATE_RECORDS
);

foreach ($robots as $robot) {
    echo $robot->year, PHP_EOL;
}
```
Hydration mode也可以作为`find()`方法的参数传递：
```php
<?php

use Phalcon\Mvc\Model\Resultset;
use Store\Toys\Robots;

$robots = Robots::find(
    [
        'hydration' => Resultset::HYDRATE_ARRAYS,
    ]
);

foreach ($robots as $robot) {
    echo $robot['year'], PHP_EOL;
}
```
## 表前缀(Table prefixes)
如果希望所有表名称都有特定前缀，并且不想在所有模型中都调用`setSource()`方法，则可以调用`Phalcon\Mvc\Model\Manager`的`setModelprefix()`方法：
```php
<?php

use Phalcon\Mvc\Model;
use Phalcon\Mvc\Model\Manager;

class Robots extends Model
{

}

$manager = new Manager();
$manager->setModelPrefix('wp_');
$robots = new Robots(null, null, $manager);
echo $robots->getSource(); // 返回wp_robots
```
## 自动生成的标识字段(Auto-generated identity columns)
某些模型有标识字段，这些字段通常是映射表的主键。`Phalcon\Mvc\Model`能够识标识字段，并在生成INSERT语句时忽略它，所以数据库能够自动为它生成一个值。创建记录之后，标识字段的值会被注册为数据库为其生成的值:
```php
<?php

$robot->save();

echo 'The generated id is: ', $robot->id;
```
`Phalcon\Mvc\Model`能够识别标识字段，根据数据库系统，这些字段可能PostgreSQL的串行列，或者是MySQL的自增列。

PostgreSQL使用序列生成自增值，默认情况下，Phalcon试图从序列`table_field_seq`中获取生成的值，例如：`robots_id_seq`，如果序列具有其他名称，则需要实现`getSequenceName()`方法：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function getSequenceName()
    {
        return 'robots_sequence_name';
    }
}
```
## 忽略字段(Skipping Columns)
为`Phalcon\Mvc\Model`指定创建 / 更新记录时需要被忽略的字段，以便数据库为其赋默认值：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        // INSERT / UPDATE操作均忽略字段
        $this->skipAttributes(
            [
                'year',
                'price',
            ]
        );

        // INSERT操作忽略字段
        $this->skipAttributes(
            [
                'created_at',
            ]
        );

        // UPDATE操作忽略字段
        $this->skipAttributes(
            [
                'modified_in',
            ]
        );
    }
}
```
这将全局忽略整个应用程序中每个INSERT / UPDATE操作的这些字段。如果想在不同的INSERT / UPDATE操作时忽略不同字段，可以传递第二个参数(布尔值) - true。强制使用默认值，实现方式如下：
```php
<?php

use Phalcon\Db\RawValue;
use Store\Toys\Robots;

$robot = new Robots();

$robot->name       = 'Bender';
$robot->year       = 1999;
$robot->created_at = new RawValue('default');

$robot->create();
```
回调函数也可以用于为默认值创建分配条件：
```php
<?php

namespace Store\Toys;

use Phalcon\Db\RawValue;
use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function beforeCreate()
    {
        if ($this->price > 10000) {
            $this->type = new RawValue('default');
        }
    }
}
```
切勿使用`Phalcon\Db\RawValue`传递外部数据(如用户输入)或可变数据，因为参数绑定时，这些字段的值也会被忽略，所以有可能会被用来实施注入攻击。
## 动态更新(Dynamic Updates)
UPDATE语句默认使用模型中定义的所有字段创建(全字段更新SQL)，可以更改特定模型以进行动态更新，只用有更新的字段创建SQL语句。
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        $this->useDynamicUpdate(true);
    }
}
```
## 独立列映射
ORM支持独立的列映射，它允许开发者在模型中定义与映射表列名称不相同的字段名，Phalcon会识别新的字段名称，并重命名字段以匹配数据库中相应的列。这是一项很棒的功能，当需要重命名数据库中的列名称时，不用为需要更改所有查询代码而担心，模型中的映射列会处理好这一切。例如：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public $code;

    public $theName;

    public $theType;

    public $theYear;

    public function columnMap()
    {
        return [
            'id'       => 'code',
            'the_name' => 'theName',
            'the_type' => 'theType',
            'the_year' => 'theYear',
        ];
    }
}
```
然后，你可以很自然的使用新的字段名:
```php
<?php

use Store\Toys\Robots;

// 根据name查找一个记录
$robot = Robots::findFirst(
    'theName = "Voltron"'
);

echo $robot->theName, "\n";

// 根据type排序
$robot = Robots::find(
    [
        'order' => 'theType DESC',
    ]
);

foreach ($robots as $robot) {
    echo 'Code: ', $robot->code, "\n";
}

// 添加记录
$robot = new Robots();

$robot->code    = '10101';
$robot->theName = 'Bender';
$robot->theType = 'Industrial';
$robot->theYear = 2999;

$robot->save();
```
重命名字段时需要注意以下事项：

- 关系 / 验证器中对属性的引用必须使用新名称
- 引用实际列名称将导致ORM异常

独立列映射允许你：

- 使用自定义约定编写应用程序
- 清除代码中列的前后缀
- 改变列名称而无需更改应用代码

## 记录快照(Record Snapshots)
查询时可以设置特定的模型以保持记录快照。可以使用此功能来实现审计，或是根据持久性查询的数据了解哪些字段发生了改变：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        $this->keepSnapshots(true);
    }
}
```
当激活此同能时，应用程序消耗更多的内存来与持久性查询的原始数据保持同步。在激活此功能的模型中，你可以按如下方式检查发生改变的字段：
```php
<?php

use Store\Toys\Robots;

// 查找一条记录
$robot = Robots::findFirst();

// 改变列值
$robot->name = 'Other name';

var_dump($robot->getChangedFields()); // ['name']

var_dump($robot->hasChanged('name')); // true

var_dump($robot->hasChanged('type')); // false
```
快照会在模型创建 / 更新时更新，使用`hasUpdated()`方法和`getUploadedFields()`方法检查create / save / update操作后，字段是否更新。但是在`afterUpdate()`，`afterSave()`和`afterCreate()`方法中调用`getChangedFields()`方法，会导致应用程序出问题。

你可以禁用此功能：
```php
<?php

Phalcon\Mvc\Model::setup(
    [
        'updateSnapshotOnSave' => false,
    ]
);
```
也可以在`php.ini`中设置：
```ini
phalcon.orm.update_snapshot_on_save = 0
```
使用此功能会产生以下效果：
```php
<?php

use Phalcon\Mvc\Model;

class User extends Model
{
    public function initialize()
    {
        $this->keepSnapshots(true);
    }
}

$user       = new User();
$user->name = 'Test User';
$user->create();
var_dump($user->getChangedFields());
$user->login = 'testuser';
var_dump($user->getChangedFields());
$user->update();
var_dump($user->getChangedFields());
```
在Phalcon 3.1.0及之后的版本中：
```php
array(0) {
}
array(1) {
[0] => string(5) "login"
}
array(0) {
}
```
`getUpdatedFields()`方法将正确返回更新的字段，或如上所述，你可以通过设置相关的ini值回到先前的行为。
## 指向不同模式(Pointing to a different schema)
如果模型映射到非默认的模式 / 数据库，可以使用`setSchema()`方法重新定义它：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        $this->setSchema('toys');
    }
}
```
## 多数据库配置(Setting multiple databases)
Phalcon应用中，所有模型可以属于相同的数据库连接，或具有独立的数据库连接。实际上，当`Phalcon\Mvc\Model`需要连接数据库时，它会在应用程序的服务容器中请求数据库服务。可以在`initialize()`方法中设置，覆盖该服务：
```php
<?php

use Phalcon\Db\Adapter\Pdo\Mysql as MysqlPdo;
use Phalcon\Db\Adapter\Pdo\PostgreSQL as PostgreSQLPdo;

// 注册MySQL数据库服务
$di->set(
    'dbMysql',
    function () {
        return new MysqlPdo(
            [
                'host'     => 'localhost',
                'username' => 'root',
                'password' => 'secret',
                'dbname'   => 'invo',
            ]
        );
    }
);

// 注册PostgreSQL数据库服务
$di->set(
    'dbPostgres',
    function () {
        return new PostgreSQLPdo(
            [
                'host'     => 'localhost',
                'username' => 'postgres',
                'password' => '',
                'dbname'   => 'invo',
            ]
        );
    }
);
```
然后，在`initialize()`方法中为模型定义连接服务：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        $this->setConnectionService('dbPostgres');
    }
}
```
Phalcon提供了更灵活的操作，你可以定义只读连接或写连接，这对于负载均衡和主从架构的的数据库非常有用：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        $this->setReadConnectionService('dbSlave');

        $this->setWriteConnectionService('dbMaster');
    }
}
```
ORM还支持分片功能，允许你根据当前查询条件实施分片选择：
