# 概览(Overview)
## 使用控制器(Using Controllers)
Actions是控制器中用于处理请求的方法。默认情况下，控制器中所有公共方法都映射到Actions，能够通过URL访问。Actions负责解释请求并创建响应，响应通常以视图形式呈现，或通过其他方式创建。

当访问类似`http://localhost/blog/posts/show/2015/the-post-title`的URL时，Phalcon会像下面这样解析URL的各个部分：

<table>
    <tbody>
        <tr>
            <td>Phalcon目录</td>
            <td>blog</td>
        </tr>
        <tr>
            <td>控制器</td>
            <td>posts</td>
        </tr>
        <tr>
            <td>方法</td>
            <td>show</td>
        </tr>
        <tr>
            <td>参数</td>
            <td>2015</td>
        </tr>
        <tr>
            <td>参数</td>
            <td>the-post-title</td>
        </tr>
    </tbody>
</table>

这种情况下，控制器`PostsController`将负责处理该请求。控制器可以通过`Phalcon\Loader`加载，因此控制器存放在应用中什么地方，并没有强制要求，可以根据需求自由的组织控制器。

控制器名称以`Controller`结尾，Actions名称以`Action`结尾。
```php
<?php

use Phalcon\Mvc\Contrller;

class PostsController extends Contrller
{
    public function indexAction()
    {

    }

    public function showAction($year, $postTitle)
    {

    }
}
```
 额外的URI参数被定义为Action的参数，可以通过局部变量访问它们。控制器如果继承基类`Phalcon\Mvc\Controller`，便可以访问应用中的各种服务。

 没有默认值的参数被视为必选参数，可以像PHP那样为参数设定默认值：
```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function indexAction()
    {

    }

    public function showAction($year = 2015, $postTitle = 'some default title')
    {

    }
}
```
参数按照它们在路由中传递的顺序进行分配，可以通过参数名称获取任意参数：
```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function indexAction()
    {

    }

    public function showAction()
    {
        $year      = $this->dispatcher->getParam('year');
        $postTitle = $this->dispatcher->getParam('postTitle');
    }
}
```
## 调度循环(Dispatch Loop)
调度循环在调度器中运行，直到没有剩余操作需要执行。上例中，只有一个动作被执行。`forward()`方法在调度循环中提供更复杂的操作流，可以将操作转发给其他控制器 / 方法。
```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function indexAction()
    {

    }

    public function showAction($year, $postTitle)
    {
        $this->flash->error(
            "You don't have permission to access this area"
        );

        // 转发给另一个方法
        $this->dispatcher->forwar(
            [
                'controller' => 'users',
                'action'     => 'signin',
            ]
        );
    }
}
```
如果用户没有访问某个方法的权限，则将用户转发到`UsersController`控制器的`signin`方法。
```php
<?php

use Phalcon\Mvc\Controller;

class UsersController extends Controller
{
    public function indexAction()
    {

    }

    public function signinAction()
    {

    }
}
```
## 初始化控制器(Initializing Controllers)
`Phalcon\Mvc\Controller`提供了`initialize()`方法，它在所有方法被执行前执行，不建议使用构造方法`__construct()`。
```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public $settings;

    public function initialize()
    {
        $this->settings = [
            'mySetting' => 'value',
        ];
    }

    public function saveAction()
    {
        if ($this->settings['mySetting'] === 'value') {
            // ...
        }
    }
}
```
只有当`beforeExecuteRoute`事件成功执行时，`initialize()`方法才被调用，避免了初始化方法中的应用逻辑无法在未授权的情况下执行。

如果想在构造控制器对象之后执行初始化逻辑，可以实现`onConstruct()`方法：
```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function onConstruct()
    {
        // ...
    }
}
```
注意，即使被调用的方法不存在于控制器中，或者用户无权访问(根据开发者定义的权限控制)该方法，`onConstruct()`方法仍会被执行。
## 注入服务(Injecting Services)
继承了`Phalcon\Mvc\Controller`的控制器，可以访问应用中的服务容器。例如，如果注册了这样的服务：
```php
<?php

use Phalcon\Di;

$di = new Di();

$di->set(
    'storage',
    function () {
        return new Storage(
            '/some/directory'
        );
    },
    true
);
```
可以通过多种方式访问该服务：
```php
<?php

use Phalcon\Mvc\Controller;

class FilesController extends Controller
{
    public function saveAction()
    {
        // 访问与服务同名的属性来注入服务
        $this->storage->save('/some/file');

        // 从DI中访问服务
        $this->di->get('storage')->save('/some/file');

        // 使用魔术方法getter
        $this->di->getStorage()->save('/some/file');
        $this->getDi()->getStorage()->save('/some/file');

        // 使用数组语法
        $this->di['storage']->save('/some/file');
    }
}
```
## 请求和响应(Request and Response)
假设框架预先注册好了服务。`request`服务包含一个`Phalocn\Http\Request`实例，`response`服务包含一个`Phalcon\Http\Response`实例，表示将要发送给客户端的内容。
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
响应对象通常不是直接被使用，而是在方法执行前构建。有时，比如`afterDispatch`事件中，直接访问响应对象很有用：
```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function indexAction()
    {

    }

    public function notFoundAction()
    {
        // 发送HTTP 404 响应头
        $this->response->setStatusCode(404, 'Not Found');
    }
}
```
## Session数据(Session Data)
Session能够在请求之间维持持久的数据，可以从任何控制器中访问`Phalcon\Session\Bag`来封装需要持久化的数据：
```php
<?php

use Phalcon\Mvc\Controller;

class UserController extends Controller
{
    public function indexAction()
    {
        $this->persistent->name = 'Micheal';
    }

    public function welcomeAction()
    {
        echo 'Welcome, ', $this->persistent->name;
    }
}
```
## 服务充当控制器(Using Services as Controller)
服务可以充当控制器，控制器总是从服务容器中请求。因此，以类名称注册的任何服务，都可以充当控制器角色：
```php
<?php

// 将控制器注册为服务
$di->set(
    'IndexController',
    function () {
        $component = new Component();

        return $component;
    }
);

// 带命名空间的控制器
$di->set(
    'Backend\Controllers\IndexController',
    function () {
        $component = new Component();

        return $component;
    }
);
```
## 控制器事件(Events in Controllers)
控制器自动监听调度事件，实现与事件名称同名的方法，可以在操作执行之前 / 之后实现钩子：
```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function beforeExecuteRoute($dispatcher)
    {
        // 在所有动作之前执行
        if ($dispatcher->getActionName() === 'save') {
            $this->flash->error(
                "You don't have permission to save posts"
            );

            $this->dispatcher->forward(
                [
                    'controller' => 'home',
                    'action'     => 'index',
                ]
            );
        }
    }

    public function afterExecuteRoute($dispatcher)
    {
        // 在所有动作之后执行
    }
}
```
