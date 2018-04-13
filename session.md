# Session(Storing data in the Session)
session组件提供面向对象的方式访问session数据。

使用session组件替代原生session的原因：
- 可以隔离同一域名下不同应用间的session数据
- 能够侦听应用中何处获取 / 设置了session数据
- 根据需要更改session适配器
## 启用Session(Starting the Session)
```php
<?php

use Phalcon\Session\Adapter\Files as Session;

// 某个组件请求session服务时，session开启
$di->setShared(
    'session',
    function () {
        $session = new Session();

        $session->start();

        return $session;
    }
);
```
## 工厂类(Factory)
使用适配器选项加载Session适配器：
```php
<?php

use Phalcon\Session\Factory;

$options = [
    'uniqueId'   => 'my-private-app',
    'host'       => '127.0.0.1',
    'port'       => 11211,
    'persistent' => true,
    'lifetime'   => 3600,
    'prefix'     => 'my_',
    'adapter'    => 'memcache',
];

$session = Factory::load($options);
```
## 存储 / 获取Session数据(Storing / Retrieving data in Session)
从控制器、视图或继承`Phalcon\Di\Injectable`的组件中，可以使用session服务存取session数据：
```php
<?php

use Phalcon\Mvc\Controller;

class UserController extends Controller
{
    public function indexAction()
    {
        // 设置session
        $this->session->set('user-name', 'Micheal');
    }

    public function welcomeAction()
    {
        // 检查session是否设置
        if ($this->session->has('user-name')) {
            // 获取该值
            $name = $this->session->get('user-name');
        }
    }
}
```
## 删除 / 销毁Sessions(Removing / Destroying Sessions)
```php
<?php

use Phalcon\Mvc\Controller;

class UserController extends Controller
{
    public function removeAction()
    {
        // 删除session
        $this->session->remove('user-name');
    }

    public function logoutAction()
    {
        // 销毁整个session
        $this->session->destroy();
    }
}
```
## 隔离应用间的Session数据(Isolating Session Data between Applications)
有时，用户会多次访问同一服务器下同一session的同一应用。使用session变量时，我们希望每个应用程序都有单独的session数据(即使代码相同，变量名称相同)。要解决这个问题，可以为应用中创建的每个session变量添加前缀：
```php
<?php

use Phalcon\Session\Adapter\Files as Session;

// 隔离session数据
$di->set(
    'session',
    function () {
        // 使用前缀
        $session = new Session(
            [
                'uniqueId' => 'my-app-1',
            ]
        );

        return $session;
    }
);
```
参数uniqueId是可选的。
## Session包(Session Bags)
`Phalcon\Session\Bag`组件能将session数据按照命名空间进行隔离，在`bag`中设置的变量，会自动存储到session中：
```php
<?php

use Phalcon\Session\Bag as SessionBag;

$user = new SessionBag('user');

$user->setDi($di);

$user->name = 'Kimbra Johnson';
$user->age  = 22;
```
## 持久数据(Persistent Data in Components)
控制器、组件和继承`Phalcon\Di\Injectable`的类都注入了`Phalcon\Session\Bag`组件，该组件隔离每个类的变量，因此可以独立的为每个类保存数据。
```php
<?php

use Phalcon\Mvc\Controller;

class UserController extends Controller
{
    public function indexAction()
    {
        // 创建持久变量'name'
        $this->persistent->name = 'Laura';
    }

    public function welcomeAction()
    {
        if (isset($this->persistent->name)) {
            echo 'Welcom, ', $this->persistent->name;
        }
    }
}
```
在组件当中使用它：
```php
<?php

use Phalcon\Mvc\User\Component;

class Security extends Component
{
    public function auth()
    {
        // 创建持久变量'name'
        $this->persistent->name = 'Laura';
    }

    public function getAuthName()
    {
        return $this->persistent->name;
    }
}
```
添加到session(`$this->session`)中的数据可以在整个应用访问，而添加到persistent(`$this->persistent`)中的数据只能在当前类中访问。
## 自定义适配器(Implementing your own adapters)
创建新适配器或扩展现有适配器都必须实现接口`Phalcon\Session\AdapterInterface`。