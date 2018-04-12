# Cookies管理(Cookies Management)
Cookies可以在客户端存储少量数据，即使用户关闭了浏览器。`Phalcon\Http\Response\Cookies`是一个全局cookies包，它在请求执行期间存储cookies，并在请求结束时发送cookies。
## 基础用法(Basic Usage)
在应用程序中访问`cookies`服务，即可设置 / 获取cookies：
```php
<?php

use Phalcon\Mvc\Controller;

class SessionController extends Controller
{
    public function loginAction()
    {
        // 检查cookie是否存在
        if ($this->cookies->has('remember-me')) {
            // 获取cookie
            $rememberMeCookie = $this->cookies->get('remember-me');

            // 设置cookie值
            $value = $rememberMeCookie->getValue();
        }
    }

    public function startAction()
    {
        $this->cookies->set(
            'remember-me',
            'some value',
            time() + 15 * 86400
        );
    }

    public function logoutAction()
    {
        $rememberMeCookie = $this->cookies->get('remember-me');

        // 删除cookie
        $rememberMeCookie->delete();
    }
}
```
## Cookies加密 / 解密(Encryption / Decryption of Cookies)
cookies被发送给客户端之前，会被自动加密，被用户检索时又会被自动解密，这样可以防止未经授权的用户在客户端查看cookies内容。尽管存在这种保护机制，仍不应该在cookies中存储敏感数据。

可以按如下方式禁用加密：
```php
<?php

use Phalcon\Http\Response\Cookies;

$di->set(
    'cookies',
    function () {
        $cookies = new Cookies();

        $cookies->useEncryption(false);

        return $cookies;
    }
);
```
如果需要使用加密，必须在`crypt`服务中设置全局密钥：
```php
<?php

use Phalcon\Crypt;

$di->set(
    'crypt',
    function () {
        $crypt = new Crypt();

        $crypt->setKey('#1dj8$=dp?.ak//j1V$'); // 使用自己的密钥

        return $crypt;
    }
);
```
向客户端发送未加密的cookies数据(复杂对象，结果集，服务信息等)，有可能暴露应用程序内部信息。如果不使用加密，建议仅在cookies中存储基本数据，如数字或短字符串。
