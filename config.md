# 读取配置(Reading Configurations)
`Phalcon\Config`组件用于将各种格式的配置文件转换为PHP对象，以供应用程序使用。

可以按如下方式，从`Phalcon\Config`中获取配置项：
```php
<?php

use Phalcon\Config;

$config = new Config(
    [
        'test' => [
            'parent' => [
                'property'  => 1,
                'property2' => 'yeah',
            ],
        ],
    ]
);

echo $config->get('test')->get('parent')->get('property');
echo $config->test->parent->property;
echo $config->path('test.parent.property');
```
## 工厂类(Factory)
使用`adapter`选项加载Config适配器，如果没有提供配置文件扩展名，`adapter`的值会做为扩展名添加到`filePath`值上。
```php
<?php

use Phalcon\Config\Factory;

$options = [
    'filePath' => 'path/config',
    'adapter'  => 'php',
];

$config = Factory::load($options);
```
## 原生数组(Native Arrays)
将原生数组转换成`Phalcon\Config`对象，该方法性能最佳，因为在请求期间不需要读取文件。
```php
<?php

use Phalcon\Config;

$settings = [
    'database'  => [
        'adapter'  => 'Mysql',
        'host'     => 'localhost',
        'username' => 'scott',
        'password' => 'cheetah',
        'dbname'   => 'test_db',
    ],
    'app'       => [
        'controllersDir' => '../app/controllers/',
        'modelsDir'      => '../app/models/',
        'viewsDir'       => '../app/views/',
    ],
    'mysetting' => 'the-value',
];

$config = new Config($settings);

echo $config->app->controllersDir, "\n";
echo $config->database->username, "\n";
echo $config->mysetting, "\n";
```
## 文件适配器(File Adapters)

<table>
    <tbody>
        <tr>
            <td>
                <code>Phalcon\Config\Adapter\Ini</code>
            </td>
            <td>用INI文件存储配置，内部适配器使用PHP函数<code>parse_ini_file</code></td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Config\Adapter\Json</code>
            </td>
            <td>用JSON文件存储配置</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Config\Adapter\Php</code>
            </td>
            <td>用PHP多维数组存储配置，性能最佳</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Config\Adapter\Yaml</code>
            </td>
            <td>使用YAML文件存储配置</td>
        </tr>
    </tbody>
</table>

## 读取INI文件(Reading INI Files)
Ini文件常用来存储配置，`Phalcon\Config`使用PHP函数`parse_ini_files`读取这些文件。为便于访问，文件部分被解析为子设置。
```ini
[database]
adapter  = Mysql
host     = localhost
username = scott
password = cheetah
dbname   = test_db

[phalcon]
controllersDir = '../app/controllers/'
modelsDir      = '../app/models/'
viewsDir       = '../app/views/'

[models]
metadata.adapter = 'Memory'
```
可以按如下方式读取上述配置文件：
```php
<?php

use Phalcon\Config\Adapter\Ini as ConfigIni;

$config = new ConfigIni('path/config.ini');

echo $config->phalcon->controllersDir, "\n";
echo $config->database->username, "\n";
echo $config->models->metadata->adapter, "\n";
```
## 合并配置(Merging Configurations)
`Phalcon\Config`能够递归的将一个配置对象的属性合并到另一个配置中，新属性被添加，而已有属性会被更新。
```php
<?php

use Phalcon\Config;

$config = new Config(
    [
        'database' => [
            'host'   => 'localhost',
            'dbname' => 'test_db',
        ],
        'debug'    => 1,
    ]
);

$config2 = new Config(
    [
        'database' => [
            'dbname'   => 'production_db',
            'username' => 'scott',
            'password' => 'secret',
        ],
        'logging'  => 1,
    ]
);

$config->merge($config2);

print_r($config);
```
以上代码输入以下内容：
```php
Phalcon\Config Object
(
    [database] => Phalcon\Config Object
        (
            [host] => localhost
            [dbname] => production_db
            [username] => scott
            [password] => secret
        )
    [debug] => 1
    [logging] => 1
)
```
## 嵌套配置(Nested Configuration)
可以使用`Phalcon\Config::path`方法访问嵌套的配置项。使用该方法获取配置项时，不必考虑配置项是否存在。
```php
<?php

use Phalcon\Config;

$config = new Config(
    [
        'phalcon'  => [
            'baseuri' => '/phalcon/',
        ],
        'models'   => [
            'metadata' => 'memory',
        ],
        'database' => [
            'adapter'  => 'mysql',
            'host'     => 'localhost',
            'username' => 'user',
            'password' => 'passwd',
            'name'     => 'demo',
        ],
        'test'     => [
            'parent' => [
                'property'  => 1,
                'property2' => 'yeah',
            ],
        ],
    ]
);

// 使用点号做为分隔符
$config->path('test.parent.property2'); // yeah
$config->path('database.host', null, '.'); // localhost

$config->path('test.parent'); // Phalcon\Config

// 使用斜杠做为分隔符
// 可以指定一个默认值，如果不存在，默认值将被返回
$config->path('test/parent/property3', 'no', '/'); // no

Config::setPathDelimiter('/');
$config->path('test/parent/property2'); // yeah
```
创建方法访问嵌套配置：
```php
<?php

use Phalcon\Config;
use Phalcon\Di;

/**
 * @return mixed|Config
 */
function config()
{
    $args   = func_get_args();
    $config = Di::getDefault()->getShared(__FUNCTION__);

    if (empty($args)) {
        return $config;
    }

    return call_user_func_array([$config, 'path'], $args);
}
```
## 配置依赖注入(Injecting Configuration Dependency)
在依赖注入容器中注册配置服务，然后就可以在继承`Phalcon\Mvc\Controller`的控制器中使用`Phalcon\Config`获取配置项。在入口文件中添加下列代码：
```php
<?php

use Phalcon\Config;
use Phalcon\Di\FactoryDefault;

// 创建DI
$di = new FactoryDefault();

$di->set(
    'config',
    function () {
        $configData = require 'config/config.php';

        return new Config($configData);
    }
);
```
现在，在控制器中可以使用`config`属性访问配置项：
```php
<?php

use Phalcon\Mvc\Controller;

class MyController extends Controller
{
    private function getDatabaseName()
    {
        return $this->config->database->dbname;
    }
}
```
