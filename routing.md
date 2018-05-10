# 路由(Routing)
路由组件定义路由，映射到控制器或请求处理程序。路由负责解析URI并确定请求处理程序。路由有两种模式：MVC模式和匹配模式，前者非常适合MVC应用。
## 定义路由(Defining Routes)
`Phalcon\Mvc\Router`提供高级路由功能。MVC模式下，可以定义路由，映射需要访问的控制器 / 方法。路由定义如下：
```php
<?php

use Phalcon\Mvc\Router;

// 创建路由
$router = new Router();

// 定义路由
$router->add(
    '/admin/users/my-profile',
    [
        'controller' => 'users',
        'action'     => 'profile',
    ]
);

// 另一个路由
$router->add(
    '/admin/users/change-password',
    [
        'controller' => 'users',
        'action'     => 'changePassword',
    ]
);

$router->handle();
```
`add()`方法的第一个参数是要匹配的模式，可选，第二个参数是一组路径。上例中，如果URI是`/admin/users/my-profile`，控制器`users`的`profile`方法将被执行。记住，路由并不执行控制器和方法，它只收集这些信息以通知`Phalcon\Mvc\Dispatcher`组件需要被执行的控制器 / 方法。

一个应用程序可能有许多路径，逐个定义路由会相当繁琐。面对这种情况，可以创建更加灵活的路由：
```php
<?php

use Phalcon\Mvc\Router;

// 创建路由
$router = new Router();

// 定义路由
$router->add(
    '/admin/:controller/a/:action/:params',
    [
        'controller' => 1,
        'action'     => 2,
        'params'     => 3,
    ]
);
```
上面例子使用通配符来为多个URI创建路由，访问URL `/admin/users/a/delete/dave/301`时会产生如下效果：

<table>
    <thead>
        <tr>
            <th>控制器</th>
            <th>方法</th>
            <th>参数</th>
            <th>参数</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>users</td>
            <td>delete</td>
            <td>dave</td>
            <td>301</td>
        </tr>
    </tbody>
</table>

`add()`方法支持预定义占位符或者正则表达式模式，所有路由模式必须以斜杠(`/`)开头，正则语法与PCRE正则语法相同。注意，不需要添加正则表达式分隔符，所有路由不区分大小写。

第二个参数定义各个匹配部分如何绑定到控制器 / 方法 / 参数，各匹配部分均为占位符或圆括号括起来的子模式。上例中，第一个匹配的子模式(`:controller`)是路由的控制器部分，第二个是方法，依次类推。

<table>
    <thead>
        <tr>
            <th>占位符</th>
            <th>正则</th>
            <th>用法</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <code>/:module</code>
            </td>
            <td>
                <code>/([a-zA-Z0-9\_\-]+)</code>
            </td>
            <td>匹配由字母、数字命名的模块</td>
        </tr>
        <tr>
            <td>
                <code>/:controller</code>
            </td>
            <td>
                <code>/([a-zA-Z0-9\_\-]+)</code>
            </td>
            <td>匹配由字母、数字命名的控制器</td>
        </tr>
        <tr>
            <td>
                <code>/:action</code>
            </td>
            <td>
                <code>/([a-zA-Z0-9\_\-]+)</code>
            </td>
            <td>匹配由字母、数字命名的方法</td>
        </tr>
        <tr>
            <td>
                <code>/:params</code>
            </td>
            <td>
                <code>(/.*)*</code>
            </td>
            <td>匹配由斜杠分隔的可选参数，用在路由末尾</td>
        </tr>
        <tr>
            <td>
                <code>/:namespace</code>
            </td>
            <td>
                <code>/([a-zA-Z0-9\_\-]+)</code>
            </td>
            <td>匹配命名空间</td>
        </tr>
        <tr>
            <td>
                <code>/:int</code>
            </td>
            <td>
                <code>/([0-9]+)</code>
            </td>
            <td>匹配整数参数</td>
        </tr>
    </tbody>
</table>

控制器名称为驼峰格式，字符(`-`)和(`_`)会被移除，下一个字母被大写。例如，some_controller转换为SomeController。

使用`add()`方法添加的路由，添加顺序决定了它的权重，后添加的路由比先添加的路由权重高。所有定义的路由都以倒序遍历，直到`Phalcon\Mvc\Router`找到与给定URI匹配的路由并进行处理，剩余路由会被忽略。
### 命名参数(Parameters with Names)
下面例子演示如何为路由参数命名：
```php
<?php

$router->add(
    '/news/([0-9{4}])/([0-9]{2})/([0-9]{2})/:params',
    [
        'controller' => 'posts',
        'action'     => 'show',
        'year'       => 1, // ([0-9]{4})
        'month'      => 2, // ([0-9]{2})
        'day'        => 3, // ([0-9]{2})
        'params'     => 4, // :params
    ]
);
```
上例中，路由没有定义`controller`和`action`部分，它们被替换为固定值(`posts`和`show`)，用户不知道由哪个控制器处理请求。控制器内部，可以按如下方式访问参数：
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
        // 获取参数'year'
        $year = $this->dispatcher->getParam('year');

        // 获取参数'month'
        $month = $this->dispatcher->getParam('month');

        // 获取参数'day'
        $day = $this->dispatcher->getParam('day');

        // ...
    }
}
```
注意，参数的值是从调度器中获得的，因为调度器是最终与应用程序驱动交互的组件。此外，还有一种创建命名参数的方法：
```php
<?php

$router->add(
    '/documentation/{chapter}/{name}.{type:[a-z]+}',
    [
        'controller' => 'documentation',
        'action'     => 'show',
    ]
);
```
同样可以用调度器访问这些参数：
```php
<?php

use Phalcon\Mvc\Controller;

class DocumentationController extends Controller
{
    public function showAction()
    {
        // 获取参数'name'
        $name = $this->dispatcher->getParam('name');

        // 获取参数'type'
        $type = $this->dispatcher->getParam('type');

        // ...
    }
}
```
### 短语法(Short Syntax)
如果不喜欢使用数组定义路由，可以使用替代语法。
```php
<?php

// 短格式
$router->add(
    '/posts/{year:[0-9]+}/{title:[a-z\-]+}',
    'Posts::show'
);

// 数组格式
$router->add(
    '/posts/([0-9]+)/([a-z\-]+)',
    [
        'controller' => 'posts',
        'action'     => 'show',
        'year'       => 1,
        'title'      => 2,
    ]
);
```
### 混合语法(Mixing Array and Short Syntax)
路由中可以混合使用数组和短格式语法，这种情况下请注意，命名参数会根据其定义位置自动添加到路由路径中：
```php
<?php

// 第一个参数位置必须跳过，因为被参数'country'占据
$router->add(
    '/news/{country:[a-z]{2}}/([a-z]+)/([a-z\-]+)',
    [
        'section' => 2, // 位置从2开始
        'article' => 3,
    ]
);
```
### 模块路由(Routing to Modules)
路由中可以包含模块路径，这适合于多模块应用。定义包含模块通配符的默认路由：
```php
<?php

use Phalcon\Mvc\Router;

$router = new Router(false);

$router->add(
    '/:module/:controller/:action/:params',
    [
        'module'     => 1,
        'controller' => 2,
        'action'     => 3,
        'params'     => 4,
    ]
);
```
这种情况下，路由必须始终将模块名称当成URL的一部分。例如，URL`/admin/users/edit/sonny`被解析成下列部分：

<table>
    <thead>
        <tr>
            <th>Module</th>
            <th>Controller</th>
            <th>Action</th>
            <th>Parameter</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>admin</td>
            <td>users</td>
            <td>edit</td>
            <td>sonny</td>
        </tr>
    </tbody>
</table>

绑定指定路由到特定模块：
```php
<?php

$router->add(
    '/login',
    [
        'module'     => 'backend',
        'controller' => 'login',
        'action'     => 'index',
    ]
);

$router->add(
    '/products/:action',
    [
        'module'     => 'frontend',
        'controller' => 'products',
        'action'     => 1,
    ]
);
```
绑定路由到特定命名空间：
```php
<?php

$router->add(
    '/:namespace/login',
    [
        'namespace'  => 1,
        'controller' => 'login',
        'action'     => 'index',
    ]
);
```
命名空间 / 类名称必须分开传递：
```php
<?php

$router->add(
    '/login',
    [
        'namespace'  => 'Backend\Controllers',
        'controller' => 'login',
        'action'     => 'index',
    ]
);
```
### HTTP方法约束(HTTP Method Restrictions)
使用`add()`方法添加路由时，将为任何HTTP方法启用路由。可以将路由限制到特定的方法，这在创建RESTful应用时非常有用：
```php
<?php

// 匹配GET请求
$router->addGet(
    '/products/edit/{id}',
    'Products::edit'
);

// 匹配POST请求
$router->addPost(
    '/products/save',
    'Products::save'
);

// 匹配POST或PUT请求
$router->add(
    '/products/update',
    'Products::update'
)->via(
    [
        'POST',
        'PUT',
    ]
);
```
### 参数转换(Using conversors)
在将参数传递给调度器之前，可以自由转换路由参数。
```php
<?php

// 方法名称允许使用'-'，如/products/new-ipod-nano-4-generation

$route = $router->add(
    '/products/{slug:[a-z\-]+}',
    [
        'controller' => 'products',
        'action'     => 'show',
    ]
);

$route->convert(
    'slug',
    function ($slug) {
        // 移除slug中的'-'
        return str_replace('-', '', $slug);
    }
);
```
另一种转换方式是在路由中绑定模型，直接将模型传递给action：
```php
<?php

// 假设ID被用作url中的参数：/products/4
$route = $router->add(
    '/products/{id}',
    [
        'controller' => 'products',
        'action'     => 'show',
    ]
);

$route->convert(
    'id',
    function ($id) {
        // 获取模型
        return Product::findFirstById($id);
    }
);
```
### 路由分组(Groups of Routes)
如果一组路由具有相同的路径，可以对它们进行分组，以方便维护：
```php
<?php

use Phalcon\Mvc\Router;
use Phalcon\Mvc\Router\Group as RouterGroup;

$router = new Router();

// 使用通用模块和控制器创建分组
$blog = new RouterGroup(
    [
        'module'     => 'blog',
        'controller' => 'index',
    ]
);

// 所有路由以/blog开头
$blog->setPrefix('/blog');

// 往分组中添加路由
$blog->add(
    '/save',
    [
        'action' => 'save',
    ]
);

$blog->add(
    '/edit/{id}',
    [
        'action' => 'edit',
    ]
);

// 该路由映射到与默认值不同的控制器
$blog->add(
    '/blog',
    [
        'controller' => 'blog',
        'action'     => 'index',
    ]
);

// 添加分组到路由中
$router->mount($blog);
```
可以将路由分组放到单独的文件中，以更好的组织和重用代码：
```php
<?php

use Phalcon\Mvc\Router\Group as RouterGroup;

class BlogRoutes extends RouterGroup
{
    public function initialize()
    {
        // 默认路径
        $this->setPaths(
            [
                'module'    => 'blog',
                'namespace' => 'Blog\Controllers',
            ]
        );

        // 所有路由均以/blog开头
        $this->setPrefix('/blog');

        // 添加路由分组
        $this->add(
            '/save',
            [
                'action' => 'save',
            ]
        );

        $this->add(
            '/edit/{id}',
            [
                'action' => 'edit',
            ]
        );

        // 与默认路由不同的映射
        $this->add(
            '/blog',
            [
                'controller' => 'blog',
                'action'     => 'index',
            ]
        );
    }
}
```
然后将分组添加到路由中：
```php
<?php

// 将分组加入路由
$router->mount(
    new BlogRoutes()
);
```
## 路由匹配(Matching Routes)
有效的URI传递给路由组件，然后组件处理URI并寻找匹配路由。默认的，URI取自rewrite引擎模块创建的的`$_GET['_url']`变量，有几个rewrite规则可以很好的和Phalcon配合使用：
```php
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^((?s).*)$ index.php?_url=/$1 [QSA,L]
```
在这个配置中，任何对不存在文件或不存在目录的请求都会被发送到`index.php`，以下示例展示如何在独立模式下使用该组件：
```php
<?php

use Phalcon\Mvc\Router;

// 创建路由
$router = new Router();

// 定义路由
// ...

// 从$_GET['_url']中获取URI
$router->handle();

// 或者直接设置URI
$router->handle('/employees/edit/17');

// 获取处理控制器
echo $router->getControllerName();

// 获取处理方法
echo $router->getActionName();

// 获取匹配路由
$route = $router->getMatchedRoute();
```
## 路由命名(Naming Routes)
添加到路由的每一条路由规则，在应用内部都以`Phalcon\Mvc\Router\Route`对象形式存储，该类封装了所有路由的详细规则。例如，可以为应用中的路由设定唯一标识名称。使用路由创建URLs时，这一点特别有用。
```php
<?php

$route = $router->add(
    'posts/{year}/{title}',
    'Posts::show'
);

$route->setName('show-posts');
```
然后使用`Phalcon\Mvc\Url`组件，利用路由名称创建路由：
```php
<?php

// 返回 /posts/2012/phalcon-1-0-released
echo $url->get(
    [
        'for'   => 'show-posts',
        'year'  => '2012',
        'title' => 'phalcon-1-0-released',
    ]
);
```
## 使用示例(Usage Examples)
自定义路由示例：
```php
<?php

// 匹配'/system/admin/a/edit/7001'
$router->add(
    '/system/:controller/a/:action/:params',
    [
        'controller' => 1,
        'action'     => 2,
        'params'     => 3,
    ]
);

// 匹配'/es/news'
$router->add(
    '/([a-z]{2})/:controller',
    [
        'controller' => 2,
        'action'     => 'index',
        'language'   => 1,
    ]
);

// 匹配'/es/news'
$router->add(
    '/{language:[a-z]{2}}/:controller',
    [
        'controller' => 2,
        'action'     => 'index',
    ]
);

// 匹配'/admin/posts/edit/100'
$router->add(
    '/admin/:controller/:action/:int',
    [
        'controller' => 1,
        'action'     => 2,
        'id'         => 3,
    ]
);

// 匹配'/posts/2015/02/some-cool-content'
$router->add(
    '/posts/([0-9{4}])/([0-9]{2})/([a-z\-]+)',
    [
        'controller' => 'posts',
        'action'     => 'show',
        'year'       => 1,
        'month'      => 2,
        'title'      => 3,
    ]
);

// 匹配'/manual/en/translate.adapter.html'
$router->add(
    '/manual/([a-z]{2})/([a-z\.]+)\.html',
    [
        'controller' => 'manual',
        'action'     => 'show',
        'language'   => 1,
        'file'       => 2,
    ]
);

// 匹配'/feed/fr/le-robots-hot-news.atom'
$router->add(
    '/feed/{lang:[a-z]+}/{blog:[a-z\-]+}\.{type:[a-z\-]+}',
    'Feed::get'
);

// 匹配'/api/v1/users/peter.json'
$router->add(
    '/api/(v1|v2)/{method:[a-z]+}/{param:[a-z]+}\.(json|xml)',
    [
        'controller' => 'api',
        'version'    => 1,
        'format'     => 4,
    ]
);
```
注意控制器和命名空间的正则表达式中允许使用的字符。由于类名称由这些字符组成，它们按顺序传递给文件系统，有可能被攻击者用来访问未授权文件。安全的正则表达式为：`/([a-zA-Z0-9\_\-]+)`
## 默认行为(Default Behavior)
`Phalcon\Mvc\Router`默认提供一个简单的路由，匹配下列模式的URI：`/:controller/:action/:params`。
例如，URL`http://phalconphp.com/documentation/show/about.html`将被解析成：

<table>
    <thead>
        <tr>
            <th>Controller</th>
            <th>Action</th>
            <th>Parameter</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>documentation</td>
            <td>show</td>
            <td>about.html</td>
        </tr>
    </tbody>
</table>

如果不希望路由有默认行为，只需在创建路由时传递`false`作为第一个参数：
```php
<?php

use Phalcon\Mvc\Router;

// 不使用默认路由规则
$router = new Router(false);
```
## 设置默认路由(Setting the default route)
如果访问应用程序时没有使用任何路由，`'/'`路由用于确定使用什么路径来显示初始页面：
```php
<?php

$router->add(
    '/',
    [
        'controller' => 'index',
        'action'     => 'index',
    ]
);
```
## 未找到路径(Not Found Paths)
如果没有匹配到任何路由规则，可以定义一组需要在该情况下使用的路由：
```php
<?php

// 设置404路径
$router->notFound(
    [
        'controller' => 'index',
        'action'     => 'route404',
    ]
);
```
这通常用于404页面。
> 仅在没有创建默认路由：`$router = Phalcon\Mvc\Router(false);`时有效。
## 默认路径(Setting default paths)
可以为模块、控制器或方法定义默认值，当路由缺失时，它们会自动填充到路由中：
```php
<?php

// 设定默认值
$router->setDefaultModule('backend');
$router->setDefaultNamespace('Backend\Controllers');
$router->setDefaultController('index');
$router->setDefaultAction('index');

// 数组形式
$router->setDefaults(
    [
        'controller' => 'index',
        'action'     => 'index',
    ]
);
```
## 处理多余斜杠(Dealing with extra / trailing slashes)
路由中有时会有额外的斜杠，导致调度器返回未找到状态。可以将路由设置为自动移除结尾斜杠：
```php
<?php

use Phalcon\Mvc\Router;

$router = new Router();

// 自动移除结尾斜杠
$router->removeExtraSlashes(true);
```
或者修改特定路由规则，选择性的接受结尾斜杠：
```php
<?php

// [/]{0,1}允许路由以斜杠结尾
$router->add(
    '/{language:[a-z]{2}}/:controller[/]{0,1}',
    [
        'controller' => 2,
        'action'     => 'index',
    ]
);
```
## 匹配回调(Match Callbacks)
有时，只有符合特定条件的路由才被匹配。使用`beforeMatch()`方法可以为路由添加任意条件，如果该方法返回`false`，路由将视为不匹配：
```php
<?php

$route = $router->add(
    '/login',
    [
        'module'     => 'admin',
        'controller' => 'session',
    ]
);

$route->beforeMatch(
    function ($uri, $route) {
        // 检查是否Ajax请求
        if (isset($_SERVER['HTTP_X_REQUESTED_WITH']) && $_SERVER['HTTP_X_REQUESTED_WITH'] === 'XMLHttpRequest') {
            return false;
        }

        return true;
    }
);
```
可以在类当中重用此条件：
```php
<?php

class AjaxFilter
{
    public function check()
    {
        return isset($_SERVER['HTTP_X_REQUESTED_WITH']) && $_SERVER['HTTP_X_REQUESTED_WITH'] === 'XMLHttpRequest';
    }
}
```
使用该类替代匿名函数：
```php
<?php

$route = $router->add(
    '/get/info/{id}',
    [
        'controller' => 'products',
        'action'     => 'info',
    ]
);

$route->beforeMatch(
    [
        new AjaxFilter(),
        'check',
    ]
);
```
从Phalcon3开始，还有另外一种检查方法：
```php
<?php

$route = $router->add(
    '/login',
    [
        'module'     => 'admin',
        'controller' => 'session',
    ]
);

$route->beforeMatch(
    function ($uri, $route) {

        /**
         * @var string $uri
         * @var \Phalcon\Mvc\Router\Route $route
         * @var \Phalcon\DiInterface $this
         * @var \Phalcon\Http\Request $request
         */
        $request = $this->getShared('request');

        // 检查是否Ajax请求
        return $request->isAjax();
    }
);
```
## 主机名限制(Hostname Constraints)
路由中可以设置主机名限制，只有当主机名符合条件时，才进行路由匹配：
```php
<?php

$route = $router->add(
    '/login',
    [
        'module'     => 'admin',
        'controller' => 'session',
        'action'     => 'login',
    ]
);

$route->setHostName('admin.company.com');
```
主机名称也可以用正则：
```php
<?php

$route = $router->add(
    '/login',
    [
        'module'     => 'admin',
        'controller' => 'session',
        'action'     => 'login',
    ]
);

$route->setHostName('([a-z]+).company.com');
```
路由组中，设置适用于组内每条路由规则的主机名限制：
```php
<?php

use Phalcon\Mvc\Router\Group as RouterGroup;

// 创建路由组
$blog = new RouterGroup(
    [
        'module'     => 'blog',
        'controller' => 'posts',
    ]
);

// 主机名限制
$blog->setHostName('blog.mycompany.com');

// 所有路由以/blog开头
$blog->setPrefix('/blog');

// 默认路由
$blog->add(
    '/',
    [
        'action' => 'index',
    ]
);

// 路由组中添加一条路由
$blog->add(
    '/save',
    [
        'action' => 'save',
    ]
);

$blog->add(
    '/edit/{id}',
    [
        'action' => 'edit',
    ]
);

// 将路由组加入路由中
$router->mount($blog);
```
## URI来源(URI Sources)
默认情况下，URI信息是从`$_GET['_url']`中获取，它由Rewrite-Engine传递给Phalcon。如有需要，可以使用`$_SERVER['REQUEST_URI']`：
```php
<?php

use Phalcon\Mvc\Router;

// ...

// 默认使用$_GET['_url']
$router->setUriSource(
    Router::URI_SOURCE_GET_URL
);

// 使用$_SERVER['REQUEST_URI']
$router->setUriSource(
    Router::URI_SOURCE_SERVER_REQUEST_URI
);
```
或者手动将URI传递给`handle()`方法：
```php
<?php

$router->handle('/some/route/to/handle');
```
使用`Router::URI_SOURCE_GET_URL`会自动解码Uri，因为它基于`$_REQUEST`超全局变量。但是，目前使用`Router::URI_SOURCE_SERVER_REQUEST_URI`不会自动解码Uri。
## 测试路由(Testing your routes)
由于路由组件没有任何依赖关系，可以创建如下文件来进行路由测试：
```php
<?php

use Phalcon\Mvc\Router;

// 模拟真实URIs
$testRoutes = [
    '/',
    '/index',
    '/index/index',
    '/products',
    '/products/index/',
    '/products/show/101',
];

$router = new Router();

// 自定义路由
// ...

// 路由测试
foreach ($testRoutes as $testRoute) {
    // 处理路由
    $router->handle($testRoute);

    echo 'Testing ', $testRoute, '<br>';

    // 检查路由是否匹配
    if ($router->wasMatched()) {
        echo 'Controller: ', $router->getControllerName(), '<br>';
        echo 'Action: ', $router->getActionName(), '<br>';
    } else {
        echo "The route wasn't matched by any route<br>";
    }

    echo '<br>';
}
```
## 注解路由(Annotations Router)
该组件集成注解服务，使用此策略，可以直接在控制器中编写路由规则，而不是在注册服务时添加：
```php
<?php

use Phalcon\Mvc\Router\Annotations as RouterAnnotations;

$di['router'] = function () {
    // 使用注解路由，传递false是因为不使用默认规则
    $router = new RouterAnnotations(false);

    // 如果URI以/api/products开头，读取注解路由
    $router->addResource('Products', '/api/products');

    return $router;
};
```
注解可以通过如下方式定义：
```php
<?php

/**
 * @RoutePrefix('/api/products')
 */
class ProductsController
{
    /**
     * @Get(
     *     '/'
     * )
     */
    public function indexAction()
    {

    }

    /**
     * @Get(
     *     '/edit/{id:[0-9]+}',
     *     name='edit-robot'
     * )
     */
    public function editAction($id)
    {

    }

    /**
     * @Route(
     *     '/save',
     *     methods={'POST', 'PUT'},
     *     name='save-robot'
     * )
     */
    public function saveAction()
    {

    }

    /**
     * @Route(
     *     '/delete/{id:[0-9]+}',
     *     methods='DELETE',
     *     conversors={
     *         id='MyConversors::checkId'
     *     }
     * )
     */
    public function deleteAction($id)
    {

    }

    public function infoAction($id)
    {

    }
}
```
只有标记为有效注解的方法才能用作路由。支持如下注解：

<table>
    <thead>
        <tr>
            <th>名称</th>
            <th>说明</th>
            <th>用法</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>RoutePrefix</td>
            <td>所有路由的前缀，该注解必须放在类注释中</td>
            <td>
                <code>@RoutePrefix('/api/products')</code>
            </td>
        </tr>
        <tr>
            <td>Route</td>
            <td>将方法标记为路由，该注解必须放在方法注释中</td>
            <td>
                <code>@Route('/api/products/show')</code>
            </td>
        </tr>
        <tr>
            <td>Get</td>
            <td>将方法标记为路由，仅限GET请求</td>
            <td>
                <code>@Get('/api/products/search')</code>
            </td>
        </tr>
        <tr>
            <td>Post</td>
            <td>将方法标记为路由，仅限POST请求</td>
            <td>
                <code>@Post('/api/products/save')</code>
            </td>
        </tr>
        <tr>
            <td>Put</td>
            <td>将方法标记为路由，仅限PUT请求</td>
            <td>
                <code>@Put('/api/products/save')</code>
            </td>
        </tr>
        <tr>
            <td>Delete</td>
            <td>将方法标记为路由，仅限DELETE发请求</td>
            <td>
                <code>@Delete('/api/products/delete/{id}')</code>
            </td>
        </tr>
        <tr>
            <td>Options</td>
            <td>将方法标记为路由，仅限OPTIONS请求</td>
            <td>
                <code>@Option('/api/products/info')</code>
            </td>
        </tr>
    </tbody>
</table>

对于添加路由的注解，支持以下参数：

<table>
    <thead>
        <tr>
            <th>名称</th>
            <th>说明</th>
            <th>用法</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>methods</td>
            <td>定义路由必须满足的HTTP请求方法</td>
            <td>
                <code>@Route('/api/products/', methods={'GET', 'POST'})</code>
            </td>
        </tr>
        <tr>
            <td>name</td>
            <td>定义路由名称</td>
            <td>
                <code>@Route('/api/products', name='get-products')</code>
            </td>
        </tr>
        <tr>
            <td>paths</td>
            <td>传递给<code>Phalcon\Mvc\Router::add()</code>方法的路径数组</td>
            <td>
                <code>@Route('/posts/{id}/{slug}', paths={module='backend'})</code>
            </td>
        </tr>
        <tr>
            <td>conversors</td>
            <td>用于对参数进行转换</td>
            <td>
                <code>@Route('/posts/{id}/{slug}', conversors={id='MyConversor::getId'})</code>
            </td>
        </tr>
    </tbody>
</table>

如果在应用中使用模块，最好使用`addModuleResource()`方法：
```php
<?php

use Phalcon\Mvc\Router\Annotations as RouterAnnotations;

$di['router'] = function () {
    // 使用注解路由
    $router = new RouterAnnotations(false);

    // 如果URI以/api/products开头，读取Backend\Controllers\ProductsController注解
    $router->addModuleResource('backend', 'Products', '/api/products');

    return $router;
};
```
## 注册路由实例(Registering Router instance)
在服务注册期间使用Phalcon依赖注入注册路由，使其在控制器中可用。

在引导文件中添加以下代码：
```php
<?php

// 添加路由功能
$di->set(
    'router',
    function () {
        $router = require(__DIR__ . '/../app/config/routes.php');
        return $router;
    }
);
```
需要创建`app/config/routes.php`文件，并添加初始化代码：
```php
<?php

use Phalcon\Mvc\Router;

$router = new Router();

$router->add(
    '/login',
    [
        'controller' => 'login',
        'action'     => 'index',
    ]
);

$router->add(
    '/products/:action',
    [
        'controller' => 'products',
        'action'     => 1,
    ]
);

return $router;
```
## 自定义路由(Implementing your own Router)
使用自定义路由替换Phalcon自带路由，必须实现`Phalcon\Mvc\RouterInterface`接口。