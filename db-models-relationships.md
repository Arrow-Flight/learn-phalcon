# 模型关系(Model Relationships)
## 模型关系(Relationships between Models)
模型间有四种关系类型：一对一，一对多，多对一，多对多。可以是单向关系，也可以是多向关系，可以是一对一的简单关系，也可以是多对多的复杂关系。模型管理器负责管理这些关系的外键约束，有助于保证引用的完整性，以及方便快捷的访问关联模型记录。实现了模型关系，能够以统一的方式从每条记录访问关联模型数据。
## 单向关系(Unidirectional relationships)
单向关系指一个模型和另一个模型关联，反之则不然。
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
`get`前缀用来`find()` / `findFirst()`关联记录，根据关系类型调用`find()`或`findFirst()`方法：

<table>
    <thead>
        <tr>
            <th>关系类型</th>
            <th>说明</th>
            <th>隐式方法</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>多对一</td>
            <td>直接返回关联模型实例</td>
            <td>findFirst</td>
        </tr>
        <tr>
            <td>一对一</td>
            <td>直接返回关联模型实例</td>
            <td>findFirst</td>
        </tr>
        <tr>
            <td>一对多</td>
            <td>返回关联模型实例的集合</td>
            <td>find</td>
        </tr>
        <tr>
            <td>多对多</td>
            <td>返回关联模型实例的集合，隐式的与关联模型进行内连接</td>
            <td>(complex query)</td>
        </tr>
    </tbody>
</table>

可以使用`count`前缀返回表示关联模型计数的整数：
```php
<?php

use Store\Toys\Robots;

$robot = Robots::findFirst(2);

echo 'The robot has ', $robot->countRobotsParts(), " parts\n";
```
### 关系别名(Aliasing Relationships)
为了更好的说明别名是如何运作，先看下面的例子：

`robots_similar`表定义了哪些robots与其他robots相似：
```sql
mysql> desc robots_similar;
+-------------------+------------------+------+-----+---------+----------------+
| Field             | Type             | Null | Key | Default | Extra          |
+-------------------+------------------+------+-----+---------+----------------+
| id                | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| robots_id         | int(10) unsigned | NO   | MUL | NULL    |                |
| similar_robots_id | int(10) unsigned | NO   |     | NULL    |                |
+-------------------+------------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)
```
`robots_id`和`similar_robots_id`字段均关联模型Robots：

映射此表及其关联表的模型如下：
```php
<?php

class RobotsSimilar extends Phalcon\Mvc\Model
{
    public function initialize()
    {
        $this->belongsTo(
            'robots_id',
            'Store\Toys\Robots',
            'id'
        );

        $this->belongsTo(
            'similar_robots_id',
            'Store\Toys\Robots',
            'id'
        );
    }
}
```
由于两个关系都指向同一模型(Robots)，获取关联模型记录可能会不清晰：
```php
<?php

$robotsSimilar = RobotsSimilar::findFirst();

// 基于关联字段(robots_id)返回关联模型记录
// 由于是多对一关系，因此只返回一条记录
// 但是方法名'getRobots'似乎暗示返回记录不只一条

$robot = $robotsSimilar->getRobots();

// 如果关系指向指向同一字段，如何根据关联字段(similar_robots_id)获取关联记录
```
别名允许我们重命名这两个关系，以解决上述问题：
```php
<?php

use Phalcon\Mvc\Model;

class RobotsSimilar extends Model
{
    public function initialize()
    {
        $this->belongsTo(
            'robots_id',
            'Store\Toys\Robots',
            'id',
            [
                'alias' => 'Robot',
            ]
        );

        $this->belongsTo(
            'similar_robots_id',
            'Store\Toys\Robots',
            'id',
            [
                'alias' => 'SimilarRobot',
            ]
        );
    }
}
```
借助别名，可以轻松获取关联记录。还可以调用`getRelated()`方法，使用别名访问关联记录：
```php
<?php

$robotsSimilar = RobotsSimilar::findFirst();

// 根据关联字段(robots_id)返回关联记录
$robot = $robotsSimilar->getRobot();
$robot = $robotsSimilar->robot;
$robot = $robotsSimilar->getRelated('Robot');

// 根据关联字段(similar_robots_id)返回关联记录
$similarRobot = $robotsSimilar->getSimilarRobot();
$similarRobot = $robotsSimilar->similarRobot;
$similarRobot = $robotsSimilar->getRelated('SimilarRobot');
```
#### 魔术方法`Getters`和显式方法(Magic Getters vs. Explicit methods)
大多数带自动完成功能的IDEs和编辑器在使用魔术方法`getters`(包括方法和属性)时，无法准确区分方法和属性。为了解决这个问题，可以使用docblock类来指定可用的魔术行为，帮助IDE更好的实现自动完成功能：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

/**
 * 表robots模型类
 * @property Simple|RobotsParts[] $robotsParts
 * @method   Simple|RobotsParts[] getRobotsParts($parameter = null)
 * @method   integer              countRobotsParts()
 */
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
## 条件语句(Conditionals)
你可以根据条件语句创建关系。根据关系执行查询时，条件将自动添加到查询当中：
```php
<?php

use Phalcon\Mvc\Model;

// Companies have invoices issued to them (paid / unpaid)
// 模型Invoices
class Invoices extends Model
{

}

// 模型Companies
class Companies extends Model
{
    public function initialize()
    {
        // invoices所有关系
        $this->hasMany(
            'id',
            'Invoices',
            'inv_id',
            [
                'alias' => 'Invoices',
            ]
        );

        // Paid invoices关系
        $this->hasMany(
            'id',
            'Invoices',
            'inv_id',
            [
                'alias'  => 'InvoicesPaid',
                'params' => [
                    'conditions' => 'inv_status = "paid"',
                ],
            ]
        );

        // Unpaid invoices关系 + 参数绑定
        $this->hasMany(
            'id',
            'Invoices',
            'inv_id',
            [
                'alias'  => 'InvoicesUnpaid',
                'params' => [
                    'conditions' => 'inv_status <> :status:',
                    'bind'       => ['status' => 'unpaid'],
                ],
            ]
        );
    }
}
```
此外，从模型对象中访问模型关系时，可以使用`getRelated()`方法的第二个参数进一步对关联记录执行过滤或排序：
```php
<?php

// Unpaid Invoices
$company = Companies::findFirst(
    [
        'conditions' => 'id = :id:',
        'bind'       => ['id' => 1],
    ]
);

$unpaidInvoices = $company->InvoicesUnpaid;
$unpaidInvoices = $company->getInvoicesUnpaid();
$unpaidInvoices = $company->getRelated('InvoicesUnpaid');
$unpaidInvoices = $company->getRelated(
    'Invoices',
    ['conditions' => 'inv_status = "paid"']
);

// 排序
$unpaidInvoices = $company->getRelated(
    'Invoices',
    [
        'conditions' => 'inv_status = "paid"',
        'order'      => 'inv_created_date ASC',
    ]
);
```
## 虚拟外键(Virtual Foreign Keys)
默认情况下，模型关系不同于数据库外键，插入 / 更新字段时，如果该字段在关联模型中没有对应的有效值，Phalcon不会生成错误验证消息。定义关系时，可以添加第四个参数来改变此行为。

更改RobotsPart模型以演示此功能：
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
            'id',
            [
                'foreignKey' => true,
            ]
        );

        $this->belongsTo(
            'parts_id',
            'Parts',
            'id',
            [
                'foreignKey' => [
                    'message' => 'The part_id does not exist on the Parts model',
                ],
            ]
        );
    }
}
```
如果将`belongsTo()`关系更改为外键，插入 / 更新这些字段时，会验证关联模型上对应字段是否具有有效值。同样，更改`hasMany()`方法和`hasOne()`方法，如果关联模型正在使用当前模型记录，则该记录不允许被删除。
```php
<?php

use Phalcon\Mvc\Model;

class Parts extends Model
{
    public function initialize()
    {
        $this->hasMany(
            'id',
            'RobotsParts',
            'parts_id',
            [
                'foreignKey' => [
                    'message' => 'The part cannot be deleted because other rotobs are using it',
                ],
            ]
        );
    }
}
```
虚拟外键可以设置字段允许null值：
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
            'parts_id',
            'Parts',
            'id',
            [
                'foreignKey' => [
                    'allowNulls' => true,
                    'message'    => 'The part_id does not exists on the Parts model',
                ],
            ]
        );
    }
}
```
### 级联 / 限制行为(Cascade / Restrict actions)
充当外键角色的虚拟外键限制了记录创建 / 更新 / 删除，以保证数据完整性：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;
use Phalcon\Mvc\Model\Relation;

class Robots extends Model
{
    public $id;

    public $name;

    public function initialize()
    {
        $this->hasMany(
            'id',
            'Parts',
            'robots_id',
            [
                'foreignKey' => [
                    'action' => Relation::ACTION_CASCADE,
                ],
            ]
        );
    }
}
```
## 存储关联记录(Storing Related Recods)
魔术属性可以用于存储记录及其关联属性：
```php
<?php

// 创建artist记录
$artist = new Artists();

$artist->name    = 'Shinichi Osawa';
$artist->country = 'Japan';

// 创建album记录
$album = new Albums();

$album->name   = 'The One';
$album->artist = $artist;
$album->year   = 2008;

// 保存记录
$album->save();
```
一对多关系中，保存记录及相关记录：
```php
<?php

// 获取artist现有记录
$artist = Artists::findFirst(
    'name = "Shinichi Osawa"'
);

// 创建album记录
$album = new Albums();

$album->name   = 'The One';
$album->artist = $artist;

$songs = [];

// 创建第一条song记录
$songs[0]           = new Songs();
$songs[0]->name     = 'Star Guitar';
$songs[0]->duration = '5:54';

// 创建第二条song记录
$songs[1]           = new Songs();
$songs[1]->name     = 'Last Days';
$songs[1]->duration = '4:29';

// 传递songs数组
$album->songs = $songs;

// 保存记录
$album->save();
```
同时保存album和artist，会隐式使用事务，如果保存记录时出现错误，记录都不会被保存。消息会回传给用户，以获取错误信息。

注意：不能通过重载以下方法来添加关联记录：
- `Phalcon\Mvc\Model::beforeSave()`
- `Phalcon\Mvc\Model::beforeCreate()`
- `Phalcon\Mvc\Model::beforeUpdate()`

可以通过重载`Phalcon\Mvc\Model::save()`方法实现。
## 操作结果集(Operations over Resultsets)
如果结果集由完整对象组成，则能够以简单的方式对获取的记录执行操作：
### 更新关联记录(Updating related records)
与其这样：
```php
<?php

$parts = $robots->getParts();

foreach ($parts as $part) {
    $part->stock      = 100;
    $part->updated_at = time();

    if ($part->update() === false) {
        $messages = $part->getMessages();

        foreach ($messages as $message) {
            echo $message;
        }

        break;
    }
}
```
不如这样：
```php
<?php
$robots->getParts()->update(
    [
        'stock'      => 100,
        'updated_at' => time(),
    ]
);
```
`update()`接受一个匿名函数来过滤必须更新的记录：
```php
<?php

$data = [
    'stock'      => 100,
    'updated_at' => time(),
];

// 更新所有记录，type = 'basic'的除外
$robots->getParts()->update(
    $data,
    function ($part) {
        if ($part->type === Part::TYPE_BASIC) {
            return false;
        }

        return true;
    }
);
```
### 删除关联记录(Deleting related records)
与其这样：
```php
<?php

$parts = $robots->getParts();

foreach ($parts as $part) {
    if ($part->delete() === false) {
        $messages = $part->getMessages();

        foreach ($messages as $message) {
            echo $message;
        }

        break;
    }
}
```
不如这样：
```php
<?php

$robots->getParts()->delete();
```
`delete()`方法也接受一个匿名函数来过滤要删除的记录：
```php
<?php

// 删除stock大于等于0的记录
$robots->getParts()->delete(
    function ($part) {
        if ($part->stock < 0) {
            return false;
        }

        return true;
    }
);
```
