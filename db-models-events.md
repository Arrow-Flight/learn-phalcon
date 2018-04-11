# 模型事件(Model Events)
## 事件和事件管理器(Events and Events Manager)
模型允许实现执行插入 / 更新 / 删除操作时要触发的事件，这些事件可用于定义业务规则。以下是`Phalcon\Mvc\Model`支持的事件及其执行顺序：

<table>
    <thead>
        <tr>
            <th>操作</th>
            <th>事件名称</th>
            <th>是否终止操作</th>
            <th>说明</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Inserting</td>
            <td>afterCreate</td>
            <td>否</td>
            <td>
                仅在执行插入操作时，在数据库系统进行所需操作之后运行
            </td>
        </tr>
        <tr>
            <td>Deleting</td>
            <td>afterDelete</td>
            <td>否</td>
            <td>
                删除操作完成后运行
            </td>
        </tr>
        <tr>
            <td>Updating</td>
            <td>afterUpdate</td>
            <td>否</td>
            <td>
                仅在执行更新操作时，在数据库系统进行所需操作之后运行
            </td>
        </tr>
        <tr>
            <td>Inserting / Updating</td>
            <td>afterSave</td>
            <td>否</td>
            <td>
                在数据库系统进行所需操作后运行
            </td>
        </tr>
        <tr>
            <td>Inserting / Updating</td>
            <td>afterValidation</td>
            <td>是</td>
            <td>
                验证字段不为null / 空字符串或外键后运行
            </td>
        </tr>
        <tr>
            <td>Inserting</td>
            <td>afterValidationOnCreate</td>
            <td>是</td>
            <td>
                在进行插入操作时，验证字段不为null / 空字符串或外键后运行
            </td>
        </tr>
        <tr>
            <td>Updating</td>
            <td>afterValidationOnUpdate</td>
            <td>是</td>
            <td>
                在进行更新操作时，验证字段不为null / 空字符串或外键后运行
            </td>
        </tr>
        <tr>
            <td>Inserting / Updating</td>
            <td>beforeValidation</td>
            <td>是</td>
            <td>
                验证字段不为null / 空字符串或外键前运行
            </td>
        </tr>
        <tr>
            <td>Inserting</td>
            <td>beforeCreate</td>
            <td>是</td>
            <td>
                仅在执行插入操作时，在数据库系统进行所需操作之前运行
            </td>
        </tr>
        <tr>
            <td>Deleting</td>
            <td>beforeDelete</td>
            <td>是</td>
            <td>
                删除操作完成之前运行
            </td>
        </tr>
        <tr>
            <td>Inserting / Updating</td>
            <td>beforeSave</td>
            <td>是</td>
            <td>
                在数据库系统进行所需操作前运行
            </td>
        </tr>
        <tr>
            <td>Updating</td>
            <td>beforeUpdate</td>
            <td>是</td>
            <td>
                在执行更新操作时，在数据库系统进行所需操作之前运行
            </td>
        </tr>
        <tr>
            <td>Inserting</td>
            <td>beforeValidationOnCreate</td>
            <td>是</td>
            <td>
                在进行插入操作时，在验证字段不为null / 空字符串或外键之前运行行
            </td>
        </tr>
        <tr>
            <td>Updating</td>
            <td>beforeValidationOnUpdate</td>
            <td>是</td>
            <td>
                在进行更新操作时，在验证字段不为null / 空字符串或外键之前运行
            </td>
        </tr>
        <tr>
            <td>Inserting / Updating</td>
            <td>onValidationFails</td>
            <td>是(已停用)</td>
            <td>
                完整性验证失败后运行
            </td>
        </tr>
        <tr>
            <td>Inserting / Updating</td>
            <td>validation</td>
            <td>是</td>
            <td>
                在进行更新操作时，在验证字段不为null / 空字符串或外键之前运行
            </td>
        </tr>
    </tbody>
</table>

### 模型中定义事件(Implementing Events in the Model's class)
让模型对事件作出反应的简单方法，实在模型中实现与事件同名的方法：
```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function beforeValidationOnCreate()
    {
        echo 'This is a executed before creating a Robot!';
    }
}
```
在执行操作前，可以使用事件为字段赋值，例如：
```php
<?php

use Phalcon\Mvc\Model;

class Products extends Model
{
    public function beforeCreate()
    {
        // Set the creation date
        $this->created_at = date('Y-m-d H:i:s');
    }

    public function beforeUpdate()
    {
        // Set the modification date
        $this->modified_in = date('Y-m-d H:i:s');
    }
}
```
### 使用自定义事件管理器(Using a custom Events Manager)
该组件集成了`Phalcon\Events\Manager`，意味着我们可以创建事件监听：
```php
<?php

namespace Store\Toys;

use Phalcon\Events\Event;
use Phalcon\Events\Manager as EventsManager;
use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        $eventsManager = new EventsManager();
        // 使用匿名函数监听模型事件
        $eventsManager->attch(
            'model:beforeSave',
            function (Event $event, $robot) {
                if ($robot->name === 'Scooby Doo') {
                    echo "Scooby Doo isn't a robot!";

                    return false;
                }

                return true;
            }
        );

        // 为事件添加事件管理器
        $this->setEventsManager($eventManager);
    }
}
```
上例中，事件管理器仅充当对象和监听(匿名函数)之间的桥梁。保存`robots`时，