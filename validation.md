# 验证(Validation)
`Phalcon\Validation`是一个独立的验证组件，用于验证任意一组数据。该组件可用于对不属于模型或集合的对象实施验证。
```php
<?php

use Phalcon\Validation;
use Phalcon\Validation\Validator\Email;
use Phalcon\Validation\Validator\PresenceOf;

$validation = new Validation();

$validation->add(
    'name',
    new PresenceOf(
        [
            'message' => 'The name is required',
        ]
    )
);

$validation->add(
    'email',
    new PresenceOf(
        [
            'message' => 'The e-mail is required',
        ]
    )
);

$validation->add(
    'email',
    new Email(
        [
            'message' => 'The e-mail is not valid',
        ]
    )
);

$messages = $validation->validate($_POST);

if (count($messages)) {
    foreach ($messages as $message) {
        echo $message, '<br>';
    }
}
```
## 初始化验证器(Initializing Validation)
只需将验证程序添加到`Phalcon\Validation`对象，即可直接初始化验证链。可以将验证器放在单独的文件中，以便更好的组织和重用代码：
```php
<?php

use Phalcon\Validation;
use Phalcon\Validation\Validator\Email;
use Phalcon\Validation\Validator\PresenceOf;

class MyValidation extends Validation
{
    public function initialize()
    {
        $this->add(
            'name',
            new PresenceOf(
                [
                    'message' => 'The name is required',
                ]
            )
        );

        $this->add(
            'email',
            new PresenceOf(
                [
                    'message' => 'The e-mail is required',
                ]
            )
        );

        $this->add(
            'email',
            new Email(
                [
                    'message' => 'The e-mail is not valid',
                ]
            )
        );
    }
}
```
然后初始化并使用验证器：
```php
<?php

$validation = new MyValidation();

$messages = $validation->validate($_POST);

if (count($messages)) {
    foreach ($messages as $message) {
        echo $message, '<br>';
    }
}
```
## 内置验证器(Validators)

<table>
    <tbody>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Alnum</code>
            </td>
            <td>验证字段值是否仅为字母、数字字符</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Alpha</code>
            </td>
            <td>验证字段值是否仅为字母字符</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Date</code>
            </td>
            <td>验证字段值是否为有效日期</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Digit</code>
            </td>
            <td>验证字段值是否仅为数字字符</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\File</code>
            </td>
            <td>验证字段值是否为正确的文件</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Uniqueness</code>
            </td>
            <td>验证模型中的字段值是否唯一</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Numericality</code>
            </td>
            <td>验证字段值是否为有效数字</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\PresenceOf</code>
            </td>
            <td>验证字段值是否为null或空字符串</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Identical</code>
            </td>
            <td>验证字段值是否等于指定值</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Email</code>
            </td>
            <td>验证字段值是否为有效电子邮箱格式</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\ExclusionIn</code>
            </td>
            <td>验证字段值是否不在可能的值列表中</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\InclusionIn</code>
            </td>
            <td>验证字段值是否在可能的值列表中</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Regex</code>
            </td>
            <td>验证字段值是否匹配正则</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\StringLength</code>
            </td>
            <td>验证字符串长度</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Between</code>
            </td>
            <td>验证值是否介于两值之间</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Confirmation</code>
            </td>
            <td>验证值是否等于另一个值</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Url</code>
            </td>
            <td>验证字段值是否包含有效URL</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\CreditCard</code>
            </td>
            <td>验证信用卡号</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Validation\Validator\Callback</code>
            </td>
            <td>使用回调进行验证</td>
        </tr>
    </tbody>
</table>

以下示例说明如何为该组件创建额外验证程序：
```php
<?php

use Phalcon\Validation;
use Phalcon\Validation\Message;
use Phalcon\Validation\Validator;

class IpValidator extends Validator
{
    /**
     * 执行验证
     * @param  Validation $validator
     * @param  string     $attribute
     * @return boolean
     */
    public function validate(Validation $validator, $attribute)
    {
        $value = $validator->getValue($attribute);

        if (!filter_var($value, FILTER_VALIDATE_IP, FILTER_FLAG_IPV4 | FILTER_FLAG_IPV6)) {
            $message = $this->getOption('message');

            if (!$message) {
                $message = 'The IP is not valid';
            }

            $validator->appendMessage(
                new Message($message, $attribute, 'Ip')
            );

            return false;
        }

        return true;
    }
}
```
验证器必须返回一个有效的布尔值，表明验证是否成功。
## 回调验证器(Callback Validator)
通过使用`Phalcon\Validation\Validator\Callback`，可以执行自定义函数(必须返回布尔值或新的验证器类)验证字段。返回`true`表示验证成功，返回`false`则表示验证失败。当执行该验证器时，Phalcon会向验证器传递数据。
```php
<?php

use Phalcon\Validation;
use Phalcon\Validation\Validator\Callback;
use Phalcon\Validation\Validator\PresenceOf;

$validation = new Validation();
$validation->add(
    'amount',
    new Callback(
        [
            'callback' => function ($data) {
                return $data['amount'] % 2 == 0;
            },
            'message'  => 'Only even number of products are accepted',
        ]
    )
);
$validation->add(
    'amount',
    new Callback(
        [
            'callback' => function ($data) {
                if ($data['amount'] % 2 == 0) {
                    return $data['amount'] != 2;
                }

                return true;
            },
            'message'  => "You can't buy 2 products",
        ]
    )
);
$validation->add(
    'description',
    new Callback(
        [
            'callback' => function ($data) {
                if ($data['amount'] >= 10) {
                    return new PresenceOf(
                        [
                            'message' => 'You must write why you need so big amount',
                        ]
                    );
                }

                return true;
            },
        ]
    )
);

$messages = $validation->validate(['amount' => 1]); // Only even number of products are accepted
$messages = $validation->validate(['amount' => 2]); // You can't buy 2 products
$messages = $validation->validate(['amount' => 10]); // You must write why you need so big amount
```
## 验证信息(Validation Messages)
`Phalcon\Validation`的消息子系统，提供了一种灵活的方式来输出或者存储验证过程中产生的验证消息。

每条消息由一个`Phalcon\Validation\Message`实例组成，生成的消息集可以使用`getMessages()`方法获取。每条消息都包含一些扩展信息，如生成消息的属性或消息类型：
```php
<?php

$messages = $validation->validate();

if (count($messages)) {
    foreach ($messages as $message) {
        echo 'Message: ', $message->getMessage(), "\n";
        echo 'Field: ', $message->getField(), "\n";
        echo 'Type: ', $message->getType(), "\n";
    }
}
```
可以传递`message`参数来更改 / 翻译验证器的默认消息，甚至可以在消息中使用占位符`:field`代替该字段名称：
```php
<?php

use Phalcon\Validation\Validator\Email;

$validation->add(
    'email',
    new Email(
        [
            'message' => 'The e-mail is not valid',
        ]
    )
);
```
`getMessages()`方法会返回验证器产生的所有消息，可以使用`filter()`方法过滤特定字段产生的消息：
```php
<?php

$messages = $validation->validate();

if (count($messages)) {
    // 过滤'name'字段产生的消息
    $filteredMessages = $messages->filter('name');

    foreach ($filteredMessages as $message) {
        echo $message;
    }
}
```
## 数据过滤(Filtering of Data)
在验证之前过滤数据，可以确保不验证恶意数据或非法数据。
```php
<?php

use Phalcon\Validation;

$validation = new Validation();

$validation->add(
    'name',
    new PresenceOf(
        [
            'message' => 'The name is required',
        ]
    )
);

$validation->add(
    'email',
    new PresenceOf(
        [
            'message' => 'The email is required',
        ]
    )
);

// 过滤多余的空格
$validation->setFilters('name', 'trim');
$validation->setFilters('email', 'trim');
```
数据过滤和清理使用的是filter组件，可以使用内置过滤器或向此组件添加更多过滤器。
## 验证事件(Validation Events)
当验证器按类进行组织时，可以实现`beforeValidation()`和`afterValidation()`方法来执行额外的检查、过滤和清理等。如果`beforeValidation()`方法返回`false`，验证将自动取消：
```php
<?php

use Phalcon\Validation;

class LoginValidation extends Validation
{
    public function initialize()
    {
        // ...
    }

    /**
     * 验证之前执行
     * @param  array $data
     * @param  object $entity
     * @param  Phalcon\Validation\Message\Group $messages
     * @return boolean
     */
    public function beforeValidation($data, $entity, $messages)
    {
        if ($this->request->getHttpHost() !== 'admin.mydomain.com') {
            $messages->appendMessage(
                new Message('Only users can log on in the administration domain')
            );

            return false;
        }

        return true;
    }

    /**
     * 验证后执行
     * @param  array $data
     * @param  object $entity
     * @param  Phalcon\Validation\Message\Group $messages
     * @return [type]
     */
    public function afterValidation($data, $entity, $messages)
    {
        // ...添加其他消息或执行更多验证
    }
}
```
## 取消验证(Cancelling Validations)
默认情况下，所有分配给字段的验证器都会执行，即使其中有验证器验证失败。可以告诉验证组件，哪个验证器可以终止验证程序，来改变默认行为：
```php
<?php

use Phalcon\Validation;
use Phalcon\Validation\Validator\PresenceOf;
use Phalcon\Validation\Validator\Regex;

$validation = new Validation();

$validation->add(
    'telephone',
    new PresenceOf(
        [
            'message'      => 'The telephone is required',
            'cancelOnFail' => true,
        ]
    )
);

$validation->add(
    'telephone',
    new Regex(
        [
            'message' => 'The telephone is required',
            'pattern' => '/\+44 [0-9]+/',
        ]
    )
);

$validation->add(
    'telephone',
    new StringLength(
        [
            'messageMinimum' => 'The telephone is too short',
            'min'            => 2,
        ]
    )
);
```
第一个验证器的`cancelOnFail`选项设置为`true`，如果该验证器验证失败，验证链上剩下验证器将不会执行。

如果创建自定义验证程序，可以通过设置`cancelOnFail`选项来动态终止验证链。
```php
<?php

use Phalcon\Validation;
use Phalcon\Validation\Validator;

class MyValidator extends Validator
{
    /**
     * 执行验证
     * @param  Phalcon\Validation $validator
     * @param  string $attribute
     * @return boolean
     */
    public function validate(Validation $validator, $attribute)
    {
        // 属性名为'name'时，终止验证链
        if ($attribute === 'name') {
            $validator->setOption('cancelOnFail', true);
        }

        // ...
    }
}
```
## 避免空值验证(Avoid validating empty values)
可以传递选项`allowEmpty`给任意内置验证器，以避免对空值执行验证：
```php
<?php

use Phalcon\Validation;
use Phalcon\Validation\Validator\Regex;

$validation = new Validation();

$validation->add(
    'telephone',
    new Regex(
        [
            'message'    => 'The telephone is required',
            'pattern'    => '/\+44 [0-9]+/',
            'allowEmpty' => true,
        ]
    )
);
```
## 递归验证(Recursive Validation)
可以在`afterValidation()`方法里运行另一个验证器实例。在这个例子中，验证`CompanyValidation`实例时会检查`PhoneValidation`实例：
```php
<?php

use Phalcon\Validation;

class CompanyValidation extends Validation
{
    /**
     * @var PhoneValidation
     */
    protected $phoneValidation;

    public function initialize()
    {
        $this->phoneValidation = new PhoneValidation();
    }

    public function afterValidation($data, $entity, $messages)
    {
        $phoneValidationMessages = $this->phoneValidation->validate($data['phone']);

        $messages->appendMessages(
            $phoneValidationMessages
        );
    }
}
```
