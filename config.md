# 读取配置(Reading Configurations)
`Phalcon\Config`组件用于将各种格式的配置文件转换为PHP对象，以供应用程序使用。

可以按如下方式，从`Phalcon\Config`中获取值：
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
使用`adapter`选项加载Config适配器，如果没有提供配置文件扩展名，`adapter`值会添加到`filePath`值上。
```php
<?php

use Phalcon\Config\Factory;

$options = [
    'filePath' => 'path/config',
    'adapter'  => 'php',
];

$config = Factory::load($options);
```
## 本地数组(Native Arrays)
将本地数组转换成`Phalcon\Config`对象，该方法性能最佳，因为在请求期间不需要读取文件。
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
Ini文件常用来存储配置，`Phalcon\Config`使用PHP函数`parse_ini_files`读取这些文件。为便于访问，文件
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