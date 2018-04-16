# 安全(Security)
该组件帮助开发人员执行常见的安全任务，如密码散列和防护跨站请求伪造(CSRF)。
## 密码散列(Password Hashing)
以纯文本形式存储密码不安全，任何有权访问数据库的人都可以访问所有用户账户，从而可以进行未经授权的行为。为了解决这个问题，许多应用程序使用常见的单向散列算法`md5`和`sha1`。但是，硬件每天都在发展，运行速度越来越快，这些算法容易被暴力破解。这类攻击被称为'彩虹表'。

为了解决这个问题，可以使用散列算法`bcrypt`。得益于'Eksblowfish'密钥算法，可以让密码加密过程要多慢有多慢。这使得计算出散列背后的真实密码极其困难，能够长时间免受彩虹表攻击。

该组件能够以简单的方式使用此算法：
```php
<?php

use Phalcon\Mvc\Controller;

class UsersController extends Controller
{
    public function registerAction()
    {
        $user = new Users();

        $login    = $this->request->getPost('login');
        $password = $this->request->getPost('password');

        $user->login = $login;

        // 存储密码散列
        $user->password = $this->security->hash($password);

        $user->save();
    }
}
```
这里保存了使用默认加密因子加密的密码。复杂的加密因子会使密码不容易受到攻击，因为其加密速度较慢。可以按如下方式检查密码是否正确：
```php
<?php

use Phalcon\Mvc\Controller;

class SessionController extends Controller
{
    public function loginAction()
    {
        $login    = $this->request->getPost('login');
        $password = $this->request->getPost('password');

        $user = Users::findFirstByLogin($login);
        if ($user) {
            if ($this->security->checkHash($password, $user->password)) {
                // password有效
            }
        } else {
            // 为了防止时序共计，无论user是否存在，该脚本将花费大致相同的时间，因为它总会计算散列
            $this->security->hash(rand());
        }

        // 验证失败
    }
}
```
盐值由PHP函数[openssl_random_pseudo_bytes](http://php.net/manual/en/function.openssl-random-pseudo-bytes.php)使用伪随机数生成，因此需要加载openssl扩展。
## 防护跨站请求伪造(Cross-Site Request Forgery(CSRF) protection)
这是针对网站和应用程序的另一种常见攻击，表单很容易受到此攻击。

防护思路是拦截应用外部提交的表单数据。在每个表单中生成一个伪随机数(令牌)，在session中存储此令牌，然后在表单将数据发回应用程序后，通过比较session中的令牌和表单提交的令牌进行验证：
```php
<?php echo Tag::form('session/login'); ?>

    <!-- Login and password inputs ... -->

    <input type="hidden" name="<?php echo $this->security->getTokenKey(); ?>" value="<?php echo $this->security->getToken(); ?>">
</form>
```
然后在控制器方法中，检查CSRF令牌是否有效：
```php
<?php

use Phalcon\Mvc\Controller;

class SessionController extends Controller
{
    public function loginAction()
    {
        if ($this->request->isPost()) {
            if ($this->security->checkToken()) {
                // The token is OK
            }
        }
    }
}
```
记得在依赖注入中添加session适配器，否则token检查将不起作用：
```php
<?php

$di->setShared(
    'session',
    function () {
        $session = new \Phalcon\Session\Adapter\Files();

        $session->start();

        return $session;
    }
);
```
在表单中添加验证码也可以完全避免此类攻击。
## 组件设置(Setting up the component)
该组件会在服务容器中自动注册为`security`服务，可以使用其他配置重新注册：
```php
<?php

use Phalcon\Security;

$di->set(
    'security',
    function () {
        $security = new Security();

        // 将密码散列因子设置为12轮
        $security->setWorkFactor(12);

        return $security;
    },
    true
);
```
## 随机数(Random)
`Phalcon\Security\Random`类可以很轻易的生成大量随机数据。
```php
<?php

use Phalcon\Security\Random;

$random = new Random();

// ...
$bytes = $random->bytes();

// 生成长度为$len的十六进制随机字符串
$hex = $random->hex($len);

// 生成长度为$len的base64随机字符串
$base64 = $random->base64($len);

// 生成长度为$len、URL安全的base64字符串
$base64Safe = $random->base64Safe($len);

// 生成UUID(version 4)
// 参考https://en.wikipedia.org/wiki/Universally_unique_identifier
$uuid = $random->uuid();

// 生成0到$n之间的随机整数
$number = $random->number($n);
```
## 外部资源(External Resources)
- [Vökuró](https://vokuro.phalconphp.com/)是一个使用安全组件加密密码和避免CSRF攻击的应用示例，[Github](https://github.com/phalcon/vokuro)