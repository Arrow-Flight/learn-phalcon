# 响应(Returning Responses)
返回响应给客户端是HTTP循环的一部分，`Phalcon\Http\Response`是完成此项任务的Phalcon组件。HTTP响应通常由响应头和响应正文组成：
```php
<?php

use Phalcon\Http\Response;

// 获取响应实例
$response = new Response();

// 设置状态码
$response->setStatusCode(404, 'Not Found');

// 设置响应正文
$response->setContent("Sorry, the page doesn't exist");

// 发送响应给客户端
$response->send();
```
使用完整的MVC栈时，不需要手动创建响应。如果需要从控制器中直接返回响应，请参照示例：
```php
<?php

use Phalcon\Http\Response;
use Phalcon\Mvc\Controller;

class FeedController extends Controller
{
    public function getAction()
    {
        // 获取响应实例
        $response = new Response();

        $feed = // ...加载feed

        $response->setContent(
            $feed->asString()
        );

        // 返回响应
        return $response;
    }
}
```
## 响应头(Working with Headers)
响应头是HTTP响应的重要组成部分，它包含HTTP状态、响应类型等有用信息。

可以通过以下方式设置响应头：
```php
<?php

// 设置响应头
$response->setHeader('Content-Type', 'application/pdf');
$response->setHeader('Content-Disposition', 'attachment; filename="downloaded.pdf"');

// 设置原生HTTP响应头
$response->setRawHeader('HTTP/1.1 200 OK');
```
`Phalcon\Http\Response\Headers`类在管理所有响应头，它会在响应头被发送之前检索所有响应头：
```php
<?php

// 获取所有响应头
$headers = $response->getHeaders();

// 获取指定响应头
$contentType = $headers->get('Content-Type');
```
## 重定向(Making Redirections)
`Phalcon\Http\Response`可以执行HTTP重定向：
```php
<?php

// 重定向到默认URI
$response->redirect();

// 重定向到本地URI
$response->redirect('posts/index');

// 重定向到外部URL
$response->redirect('http://en/wikipedia.org', true);

// 重定向，指定HTTP状态码
$response->redirect('http://www.example.com/new-location', true, 301);
```
所有内部URI都由url服务(默认`Phalcon\Mvc\Url`)生成，下例演示如何使用指定路由进行重定向：
```php
<?php

// 重定向到指定路由
return $response->redirect(
    [
        'for'        => 'index-lang',
        'lang'       => 'jp',
        'controller' => 'index',
    ]
);
```
注意，重定向不会禁用视图组件。如果当前方法存在对应的视图，视图始终会被输出。可以在控制器中调用`$this->view->disable()`禁用视图。
## HTTP缓存
使用HTTP缓存可以提高应用性能，减小服务器访问压力，大多数现代浏览器支持HTTP缓存。

当应用程序第一次提供页面时，HTTP缓存可以在发送的响应头信息里设置：
- `Expires:`设置一个将来或过去的日期，告诉浏览器页面什么时候过期。
- `Cache-Control:`告诉浏览器，页面应该缓存多久
- `Last-Modified:`告诉浏览器，页面的最后改动时间
- `ETag:`包含当前页面最后改动时间戳的唯一标识
### 设置过期时间(Setting an Expiration Time)
从当前时间开始，添加浏览器缓存页面的时长。在有效期内，浏览器不会向服务器请求新内容：
```php
<?php

$expiryDate = new DateTime();
$expiryDate->modify('+2 months');

$response->setExpires($expiryDate);
```
响应组件自动在响应头信息中显示GMT格式的时间。

如果将该值设置为过去的时间，浏览器将始终刷新所请求的页面：
```php
<?php

$expiryDate = new DateTime();
$expiryDate->modify('-10 minutes');

$response->setExpires($expiryDate);
```
浏览器根据客户端时间判断给定时间是否过期。客户端时间可以被修改，导致缓存页面过期。
### Cache-Control
Cache-Control提供了一种更加安全的方式来缓存页面。只需指定一个秒数，告诉浏览器页面该缓存多长时间：
```php
<?php

// 页面缓存一天
$response->setHeader('Cache-Control', 'max-age=86400');
```
下列方式效果相反(避免页面缓存)：
```php
<?php

// 不缓存页面
$response->setHeader('Cache-Control', 'private, max-age=0, must-relatidate');
```
### E-Tag
`entity-tag`或`E-tag`是一个唯一标识符，帮助浏览器分析两个请求间页面是否发生改变。如果先前提供的内容已发生改变，则此标识符必须重新计算：
```php
<?php

// 根据news的最近修改时间计算E-Tag
$mostRecentDate = News::maximun(
    [
        'column' => 'created_at',
    ]
);

$eTag = md5($mostRecentDate);

// 发送E-Tag头
$response->setHeader('E-Tag', $eTag);
```
