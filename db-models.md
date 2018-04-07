# 使用模型
模型表示应用程序信息(数据)和这些数据的处理规则，主要用于管理与对应数据表的交互规则。多数情况下，数据库中的每张表都有一个模型与之对应。应用程序中的大部分业务逻辑集中在模型中。

Phalcon应用中，`Phalcon\Mvc\Model`是所有模型的基类。它提供了数据库独立，基础CRUD，高级查找，模型关联，以及其他服务。

`Phalcon\Mvc\Model`将方法动态的转换为对应的数据库操作，规避了使用SQL语句的必要。

模型使用数据库高级抽象层，如果你需要使用更为底层的方式操作数据库，请参考`Phalcon\Db`组件文档。
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
