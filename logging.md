# 日志(Logging)
`Phalcon\Logger`是为应用程序提供日志记录服务的组件，可以使用不同的适配器记录日志。支持事务日志，配置项，不同格式和过滤器。`Phalcon\Logger`可以用于应用中任何需要记录日志的地方，从过程调试到应用流程跟踪。

## 适配器(Adapters)
该组件使用适配器来存储日志消息，允许通用的日志接口，若有必要可以在后台随时切换适配器。支持如下适配器：

<table>
    <thead>
        <tr>
            <th>适配器</th>
            <th>说明</th>
        </tr>
    </thead>
    <body>
        <tr>
            <td>
                <code>Phalcon\Logger\Adapter\File</code>
            </td>
            <td>记录到纯文本文件</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Logger\Adapter\Stream</code>
            </td>
            <td>记录到PHP流</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Logger\Adapter\Syslog</code>
            </td>
            <td>记录到系统日志</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Logger\Adapter\FirePHP</code>
            </td>
            <td>记录到FirePHP</td>
        </tr>
    </body>
</table>

### 工厂类(Factory)
使用配置项加载日志适配器：
```php
<?php

use Phalcon\Logger\Factory;

$options = [
    'name'    => 'log.txt',
    'adapter' => 'file',
];

$logger = Factory::load($options);
```
## 创建日志(Creating a Log)
```php
<?php

use Phalcon\Logger;
use Phalcon\Logger\Adapter\File as FileAdapter;

$logger = new FileAdapter('app/logs/test.log');

// 可用的日志级别

$logger->critical(
    'This is a critical message'
);

$logger->emergency(
    'This is an emergency message'
);

$logger->debug(
    'This is a debug message'
);

$logger->error(
    'This is an error message'
);

$logger->info(
    'This is an info message'
);

$logger->notice(
    'This is a notice message'
);

$logger->warning(
    'This is a warning message'
);

$logger->alert(
    'This is a alert message'
);

// 使用log()方法和Logger常量
$logger->log(
    'This is another error message',
    Logger::ERROR
);

// 如果没有给出常量，默认使用DEBUG
$logger->log(
    'This is a message'
);

// 也可以像这样传递内容参数
$logger->log(
    'This is a {message}',
    [
        'message' => 'parameter',
    ]
);
```
生成的日志如下：
```php
[Tue, 28 Jul 15 22:09:02 -0500][CRITICAL] This is a critical message
[Tue, 28 Jul 15 22:09:02 -0500][EMERGENCY] This is an emergency message
[Tue, 28 Jul 15 22:09:02 -0500][DEBUG] This is a debug message
[Tue, 28 Jul 15 22:09:02 -0500][ERROR] This is an error message
[Tue, 28 Jul 15 22:09:02 -0500][INFO] This is an info message
[Tue, 28 Jul 15 22:09:02 -0500][NOTICE] This is a notice message
[Tue, 28 Jul 15 22:09:02 -0500][WARNING] This is a warning message
[Tue, 28 Jul 15 22:09:02 -0500][ALERT] This is an alert message
[Tue, 28 Jul 15 22:09:02 -0500][ERROR] This is another error message
[Tue, 28 Jul 15 22:09:02 -0500][DEBUG] This is a message
[Tue, 28 Jul 15 22:09:02 -0500][DEBUG] This is a parameter
```
还可以使用`setLogLevel()`方法设置日志级别，该方法接收一个常量，只保存级别不低于该常量的日志：
```php
<?php

use Phalcon\Logger;
use Phalcon\Logger\Adapter\File as FileAdapter;

$logger = new FileAdapter('app/logs/test.log');

$logger->setLogLevel(
    Logger::CRITICAL
);
```
上例中，只记录critical和emergency级别的日志。默认情况下，记录所有级别日志。
## 事务(Transactions)
将数据记录到文件(文件系统)中会有较高的性能损失，这种情况下可以使用日志事务。事务将日志临时存储在内存中，然后再写入相应的适配器。
```php
<?php

use Phalcon\Logger\Adapter\File as FileAdapter;

// 创建日志
$logger = new FileAdapter('app/logs/test.log');

// 开启事务
$logger->begin();

// 添加消息
$logger->alert(
    'This is an alert'
);

$Logger->error(
    'This is another error'
);

// 保存消息到文件
$logger->commit();
```
## 记录到多个处理程序(Logging to Multiple Handlers)
`Phalcon\Logger`可以通过一次调用，将日志消息发送给多个处理程序：
```php
<?php

use Phalcon\Logger;
use Phalcon\Logger\Adapter\File as FileAdapter;
use Phalcon\Logger\Adapter\Stream as StreamAdapter;
use Phalcon\Logger\Multiple as MultipleStream;

$logger = new MultipleStream();

$logger->push(
    new FileAdapter('test.log')
);

$logger->push(
    new StreamAdapter('php://stdout')
);

$Logger->log(
    'This is a message'
);

$logger->log(
    'This is an error',
    Logger::ERROR
);

$logger->error(
    'This is another error'
);
```
## 格式化消息(Messsage Formatting)
该组件在消息被发送到后台之前，使用`formatters`格式化消息。可用的`formatters`如下：

<table>
    <thead>
        <tr>
            <th>适配器</th>
            <th>说明</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <code>Phalcon\Logger\Formatter\Line</code>
            </td>
            <td>单行格式</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Logger\Formatter\FirePHP</code>
            </td>
            <td>
                FirePHP格式
            </td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Logger\Formatter\Json</code>
            </td>
            <td>JSON格式</td>
        </tr>
        <tr>
            <td>
                <code>Phalcon\Logger\Formtter\Syslog</code>
            </td>
            <td>系统格式</td>
        </tr>
    </tbody>
</table>

### 单行格式(Line Formatter)
使用单行字符串格式化消息，默认格式是：
```
[%date%][%type%] %message%
```
`setFormat()`方法能更改默认格式，可以调用该方法自定义消息格式。支持的格式变量有：

<table>
    <thead>
        <tr>
            <th>格式变量</th>
            <th>说明</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>%message%</td>
            <td>消息记录格式</td>
        </tr>
        <tr>
            <td>%date%</td>
            <td>消息添加日期</td>
        </tr>
        <tr>
            <td>%type%</td>
            <td>带消息类型的大写字符串</td>
        </tr>
    </tbody>
</table>

以下示例显示如何更改日志格式：
```php
<?php

use Phalcon\Logger\Formatter\Line as LineFormatter;

$formatter = new LineFormatter('%date% - %message%');

// 更改日志格式
$logger->setFormatter($formatter);
```
### 自定义格式(Implementing your own formatters)
扩展已有格式或创建自定义格式，必须实现`Phalcon\Logger\FormatterInterface`接口。
## 适配器(Adapters)
以下示例演示每个适配器的基本用法：
### PHP流日志(Stream Logger)
流日志将消息写入已注册的有效PHP流中。
```php
<?php

use Phalcon\Logger\Adapter\Stream as StreamAdapter;

// 使用zlib压缩开启流
$logger = new StreamAdapter('compress.zlib://week.log.gz');

// 将日志写入stderr
$logger = new StreamAdapter('php://stderr');
```
### 文本日志(File Logger)
使用文件记录日志，默认所有日志文件都使用追加模式打开，将文件指针放在文件末尾，仅用于写入。如果文件不存在，则尝试创建它。可以向构造函数传递附加参数更改此模式：
```php
<?php

use Phalcon\Logger\Adapter\File as FileAdapter;

// 以写模式创建文件
$logger = new FileAdapter(
    'app/logs/test.log',
    [
        'mode' => 'w',
    ]
);
```
### 系统日志(Syslog Logger)
将消息发送给系统日志，系统行为因操作系统而异。
```php
<?php

use Phalcon\Logger\Adapter\Syslog as SyslogAdapter;

// 基础用法
$logger = new SyslogAdapter(null);

// 配置标识、模式、设备
$logger = new SyslogAdapter(
    'ident-name',
    [
        'option'   => LOG_NDELAY,
        'facility' => LOG_MAIL,
    ]
);
```
### FirePHP日志(FirePHP Logger)
以FirePHP(Firefox的Firebug扩展)中显示的HTTP响应头格式发送消息。
```php
<?php

use Phalcon\Logger;
use Phalcon\Logger\Adapter\Firephp as Firephp;

$logger = new Firephp();

$logger->log(
    'This is a message'
);

$Logger->log(
    'This is an error',
    Logger::ERROR
);

$logger->error(
    'This is another error'
);
```
### 自定义适配器(Implementing your own adapters)
扩展已有适配器或创建自定义适配器，必须实现`Phalcon\Logger\AdapterInterface`接口。