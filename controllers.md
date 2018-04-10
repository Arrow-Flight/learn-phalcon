# 概览
## 使用控制器
Actions是控制器中用于处理请求的方法。默认情况下，控制器中所有公共方法都映射到Actions，能够通过URL访问。Actions负责解释请求并创建响应，响应通常以视图形式呈现，或通过其他方式创建。

当访问类似`http://localhost/blog/posts/show/2015/the-post-title`的URL时，Phalcon会像下面这样解析URL的各个部分：

<table>
    <thead>
        <tr>
            <th>说明</th>
            <th>模块</th>
        </tr>
    </thead>
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

控制器名称必须以`Controller`为后缀，Actions名称则必须以`Action`为后缀。
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
