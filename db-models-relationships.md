# 模型关系(Model Relationships)
## 模型关系(Relationships between Models)
模型间有四种关系类型：一对一，一对多，多对一，多对多。可以是单向关系，也可以是多向关系，可以是一对一的简单关系，也可以是多对多的复杂关系。模型管理器负责管理这些关系的外键约束，有助于保证引用的完整性，以及方便快捷的访问关联模型记录。实现了模型关系，能够以统一的方式从每条记录访问关联模型数据。
## 单向关系(Unidirectional relationships)

## 双向关系(Bidirectional relations)
双向关系指在两个模型中建立关系，每个模型中定义另一个模型的反向关系。
## 定义关系(Defining relationships)
在Phalcon中，关系必须在模型`initialize()`方法中定义，方法`belongsTo()`、`hasOne()`、`hasMany()`、`hasManyToMany()`定义了当前模型的一个或多个字段与另一个模型的字段之间的关系，每个方法有3个参数：当前模型字段，关联模型，关联字段。

<table>
    <thead>
        <tr>
            <th>方法</th>
            <th>说明</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>hasMany</td>
            <td>定义1 - n关系</td>
        </tr>
        <tr>
            <td>hasOne</td>
            <td>定义1 - 1关系</td>
        </tr>
        <tr>
            <td>belongsTo</td>
            <td>定义n - 1关系</td>
        </tr>
        <tr>
            <td>hasManyToMany</td>
            <td>定义n - n关系</td>
        </tr>
    </tbody>
</table>

下列3张关联表演示了模型间的关系：
```sql
CREATE TABLE robots (
    id int(10) unsigned NOT NULL AUTO_INCREMENT,
    name varchar(70) NOT NULL,
    type varchar(32) NOT NULL,
    year int(11) NOT NULL,
    PRIMARY KEY (id)
);

CREATE TABLE robots_parts (
    id int(10) unsigned NOT NULL AUTO_INCREMENT,
    robots_id int(10) NOT NULL,
    parts_id int(10) NOT NULL,
    created_at DATE NOT NULL,
    PRIMARY KEY (id),
    KEY robots_id (robots_id),
    KEY parts_id (parts_id)
);

CREATE TABLE parts (
    id int(10) unsigned NOT NULL AUTO_INCREMENT,
    name varchar(70) NOT NULL,
    PRIMARY KEY (id)
);
```
- 模型`Robots`与模型`RobotsParts`一对多关系
- 模型`Parts`与模型`RobotsParts`一对多关系
- 模型`RobotsParts`与模型`Robots`和模型`Parts`均为多对一关系
- 模型`Robots`通过模型`RobotsParts`与模型`Parts`成多对多关系
检查EER图以更好的理解模型关系：
模型之间的关系实现如下：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public $id;

    public $name;

    public function initialize()
    {
        $this->hasMany(
            'id',
            'RobotsParts',
            'robots_id'
        );
    }
}
```
```php
<?php

use Phalcon\Mvc\Model;

class Parts extends Model
{
    public $id;

    public $name;

    public function initialize()
    {
        $this->hasMany(
            'id',
            'RobotsParts',
            'parts_id'
        );
    }
}
```
```php
<?php

use Phalcon\Mvc\Model;

class RobotsParts extends Model
{
    public $id;

    public $robots_id;

    public $parts_id;

    public function initialize()
    {
        $this->belongsTo(
            'robots_id',
            'Store\Toys\Robots',
            'id'
        );

        $this->belongsTo(
            'parts_id',
            'Parts',
            'id'
        );
    }
}
```
第一个参数表示模型关系中，当前模型的字段；第二个参数表示关联模型的名称，第三个参数表示关联模型的字段名称。也可以使用数组来定义关系中的多个字段。

多对多关系需要3个模型，并定义关系中涉及的字段：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public $id;

    public $name;

    public function initialize()
    {
        $this->hasManyToMany(
            'id',
            'RobotsParts',
            'robots_id', 'parts_id',
            'Parts',
            'id'
        );
    }
}
```
### 使用关系(Taking advantage of relationships)
明确定义模型之间关系后，很容易查找特定记录的关联记录。
```php
<?php

use Store\Toys\Robots;

$robot = Robots::findFirst(2);

foreach ($robot->robotsParts as $robotPart) {
    echo $robotPart->parts->name, "\n";
}
```
Phalcon使用魔术方法`__set` / `__get` / `__call`存储或获取关联数据。

通过访问关系中与模型同名的属性，可以获取该模型的相关记录。
```php
<?php

use Store\Toys\Robots;

$robot = Robots::findFirst();

// RobotsParts中的关联记录
$robotsParts$robot->robotsParts;
```
也可以使用魔术方法`getter`：
```php
<?php

use Store\Toys\Robots;

$robot = Robots::findFirst();

// RobotsParts中的关联记录
$robotsParts = $robot->getRobotsParts();

// 传参
$robotsParts = $robot->getRobotsParts(
    [
        'limit' => 5,
    ]
);
```
如果被调用的方法具有`get`前缀，`Phalcon\Mvc\Model`将返回`findFirst()`或`find()`结果。下面例子比较了使用魔术方法和不使用魔术方法获取关联记录：
```php
<?php

use Store\Toys\Robots;

$robot = Robots::findFirst(2);

// 模型Robots与模型RobotsParts是一对多关系
$robotsParts = $robot->robotsParts;

// 获取符合条件的记录
$robotsParts = $robot->getRobotsParts(
    [
        'created_at = :date:',
        'bind' => [
            'date' => '2015-03-15',
        ],
    ]
);

$robotPart = RobotsParts::findFirst(1);

// 模型RobotsParts与模型Robots是多对一关系
$robot = $robotPart->robots;
```
手动获取关联记录：
```php
<?php

use Store\Toys\Robots;

$robot = Robots::findFirst(2);

// 模型Robots与模型RobotsParts是一对多关系
$robotsParts = RobotsParts::find(
    [
        'robots_id = :id:',
        'bind' => [
            'id' => $robot->id,
        ],
    ]
);

// 获取符合条件的记录
$robotsParts = RobotsParts::find(
    [
        'robots_id = :id: AND created_at = :date:',
        'bind' => [
            'id'   => $robot->id,
            'date' => '2015-03-15',
        ],
    ]
);

$robotPart = RobotsParts::findFirst(1);

// 模型RobotsParts与模型Robots是多对一关系
$robot = Robots::findFirst(
    [
        'id = :id:',
        'bind' => [
            'id' => $robotPart->robots_id,
        ],
    ]
);
```
