# 过滤和清理(Filtering and Sanitizing)
`Phalcon\Filter`组件提供了一组常用的过滤器和数据清理助手，它以面向对象的方式封装了PHP filter扩展。
## 内置过滤器(Types of Built-in Filters)
该组件内置的过滤器：

<table>
    <tbody>
        <tr>
            <td>string</td>
            <td>移除标签并将字符串转换为HTML实体，包括单引号、双引号</td>
        </tr>
        <tr>
            <td>email</td>
            <td>
                移除所有字符，字母、数字和<code>!#$%&*+-/=?^_`{|}~@.[]`除外</code>
            </td>
        </tr>
        <tr>
            <td>int</td>
            <td>移除所有字符，数字、加减号除外</td>
        </tr>
        <tr>
            <td>int!</td>
            <td>使用<code>intval</code>函数将该值转换为整型</td>
        </tr>
        <tr>
            <td>absint</td>
            <td>获取整数的绝对值</td>
        </tr>
        <tr>
            <td>float</td>
            <td>移除所有字符，数字、加减号和点号除外</td>
        </tr>
        <tr>
            <td>float!</td>
            <td>使用<code>floatval</code>函数将该值转换为浮点数</td>
        </tr>
        <tr>
            <td>alphanum</td>
            <td>移除所有字符，[a-zA-Z0-9]除外</td>
        </tr>
        <tr>
            <td>strigtags</td>
            <td>对该值应用<code>strip_tags</code>函数</td>
        </tr>
        <tr>
            <td>special_chars</td>
            <td>转义<code>'"<>&</code>和ASCII值小于32的字符</td>
        </tr>
        <tr>
            <td>trim</td>
            <td>对该值应用<code>trim</code>函数</td>
        </tr>
        <tr>
            <td>lower</td>
            <td>对该值应用<code>lower</code>函数</td>
        </tr>
        <tr>
            <td>url</td>
            <td>移除所有字符，数字、字母和<code>&#124;$`-_.+!*'(),{}[]<>#%";/?:@&=.^\\~</code>除外</td>
        </tr>
        <tr>
            <td>uppper</td>
            <td>对该值应用<code>strtoupper</code>函数</td>
        </tr>
    </tbody>
</table>

## 清理数据(Sanitizing data)
```php
<?php

use Phalcon\Filter;

$filter = new Filter();

// 返回'someone@example.com'
$filter->sanitize('some(one)@exa\mple.com', 'email');

// 返回'hello'
$filter->sanitize('hello<<', 'string');

// 返回'100019'
$filter->sanitize('!100a019', 'int');

// 返回'100019.01'
$filter->sanitize('!100a019.01a', 'float');
```
## 控制器中使用清理(Sanitizing from Controllers)
控制器中使用request对象访问`GET`或`POST`数据时，可以使用`Phalcon\Filter`对象过滤数据。第一个参数是要获取的变量名称，第二个参数是应用到变量上的过滤器。
```php
<?php

use Phalcon\Mvc\Controller;

class ProductsController extends Controller
{
    public function indexAction()
    {

    }

    public function saveAction()
    {
        // 过滤price
        $price = $this->request->getPost('price', 'double');

        // 过滤email
        $email = $this->request->getPost('customerEmail', 'email');
    }
}
```
## 过滤数据(Filtering data)
```php
<?php

use Phalcon\Filter;

$filter = new Filter();

// 返回'Hello'
$filter->sanitize('<h1>Hello</h1>', 'striptags');

// 返回'Hello'
$filter->sanitize('  Hello  ', 'trim');
```
## 过滤器组合(Combining Filters)
可以传递一组过滤器作为第二个参数，同时在一个字符串上运行多个过滤器：
```php
<?php

use Phalcon\Filter;

$filter = new Filter();

// 返回'Hello'
$filter->sanitize(
    '  <h1> Hello </h1>  ',
    [
        'striptags',
        'trim',
    ]
);
```
## 自定义过滤器(Adding filters)
可以向`Phalcon\Filter`添加自定义过滤器，过滤器可以是匿名函数：
```php
<?php

use Phalcon\Filter;

$filter = new Filter();

// 使用匿名函数
$filter->add(
    'md5',
    function ($value) {
        return preg_replace('/[^0-9a-f]/', '', $value);
    }
);

// 使用'md5'过滤器
$filtered = $filter->sanitize($possibleMd5, 'md5');
```
或者实现过滤器类：
```php
<?php

use Phalcon\Filter;

class IPv4Filter
{
    public function filter($value)
    {
        return filter_var($value, FILTER_VALIDATE_IP, FILTER_FLAG_IPV4);
    }
}

$filter = new Filter();

// 使用对象
$filter->add(
    'ipv4',
    new IPv4Filter()
);

// 使用'ipv4'过滤器
$filteredIp = $filter->sanitize('127.0.0.1', 'ipv4');
```
## 复杂过滤(Complex Sanitizing and Filtering)
PHP提供了优秀的filter扩展，可以参考PHP文档：[Data Filtering at PHP Documentation](http://www.php.net/manual/en/book.filter.php)
## 自定义Filter服务
创建自定义filter服务替换Phalcon自带服务时，必须实现接口`Phalcon\FilterInterface`。