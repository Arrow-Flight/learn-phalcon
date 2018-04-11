# 请求(Request Environment)
每个HTTP请求(通常由浏览器发起)都包含头数据、文件或变量等附加信息。Web应用需要解析这些信息，以便向请求者返回正确响应。`Phalcon\Http\Request`封装了请求信息，允许以面向对象的方式访问它。
```php
<?php

use Phalcon\Http\Request;

// 获取请求实例
$request = new Request();

// 检查是否POST请求
if ($request->isPost()) {
    // 检查是否Ajax请求
    if ($request->isAjax()) {
        echo 'Request was made using POST and AJAX';
    }
}
```
## 获取请求值(Getting Values)
PHP会根据请求类别，自动填充超全局数组`$_GET`和`$_POST`。这些数组包含表单中提交的值或通过URL传递的参数，数组中的值不会被过滤，可能包含非法字符甚至恶意代码，从而导致SQL注入或XSS攻击。

`Phalcon\Http\Request`允许访问存储在`$_REQUEST`、`$_GET`和`$_POST`中的值，并使用过滤服务(默认`Phalcon\Filter`)对它们执行清理和过滤。
```php
<?php

use Phalcon\Filter;

$filter = new Filter();

// 手动过滤
$email = $filter->sanitize($_POST['user_email'], 'email');
$email = $filter->sanitize($request->getPost('user_email'), 'email');

// 自动过滤
$email = $request->getPost('user_email', 'email');

// 如果参数为null，为其设置默认值
$email = $request->getPost('user_email', 'email', 'some@example.com');

// 如果参数为null，为其设置默认值，跳过过滤
$email = $request->getPost('user_email', null, 'some@example.com');
```
## 控制器中访问请求(Accessing the Request from Controllers)
最常访问请求的是控制器中的方法。在控制器中使用`$this->request`公共属性，可以访问`Phalcon\Http\Request`对象：
```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function indexAction()
    {

    }

    public function saveAction()
    {
        // 检查是否POST请求
        if ($this->request->isPost()) {
            // 获取POST数据
            $customerName = $this->request->getPost('name');
            $customerBorn = $this->request->getPost('born');
        }
    }
}
```
## 文件上传(Uploading Files)
`Phalcon\Http\Request`提供了一种面向对象的方式来完成文件上传：
```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function uploadAction()
    {
        // 检查用户是否上传了文件
        if ($this->request->hasFiles()) {
            $files = $this->request->getUploadedFiles();

            // 打印真实文件名和文件大小
            foreach ($files as $file) {
                // 打印文件详情
                echo $file->getName(), ' ', $file->getSize(), "\n";

                // 移动文件到应用中
                $file->moveTo(
                    'files/' . $file->getName()
                );
            }
        }
    }
}
```
## 使用请求头(Working with Headers)
请求头包含有用的信息，以便返回适当的响应给用户。
```php
<?php

// 获取Http-X-Requested-With头
$requestedWith = $request->getHeader('HTTP_X_REQUESTED_WITH');

if ($requestedWith === 'XMLHttpRequest') {
    echo 'The request was made with Ajax';
}

// 同上
if ($request->isAjax()) {
    echo 'The request was made with Ajax';
}

// 检查请求层
if ($request->isSecure()) {
    echo 'The request was made using a secure layer';
}

// 获取服务器IP，如192.168.0.100
$ipAddress = $request->getServerAddress();

// 获取客户端IP，如201.245.53.51
$ipAddress = $request->getClientAddress();

// 获取用户代理(HTTP_USER_AGENT)
$userAgent = $request->getUserAgent();

// 获取浏览器最适合内容类型，如text/xml
$contentType = $request->getAcceptableContent();

// 获取浏览器最适合的字符集，如utf-8
$charset = $request->getBestCharset();

// 获取浏览器最适合的语言，如en-us
$language = $request->getBestLanguage();

// 检查请求头是否存在
if ($request->hasHeader('my-header')) {
    echo 'Mary had a little lamb';
}
```
