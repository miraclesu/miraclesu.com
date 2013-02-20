---
title: linux下给php安装amqp扩展
date: '2013-01-26'
description: Linux下给php安装官方推荐的amqp扩展
permalink: '/2013/01/linux-php-amqp.html'
categories: [AMQP]
tags: [AMQP, PHP, Linux]
---

## 安装librabbitmq-c和rabbitmq-codegen

```
# 下载0-9-1版的rabbitmq-c
git clone git://github.com/alanxz/rabbitmq-c.git
cd rabbitmq-c
# Enable and update the codegen git submodule
git submodule init
git submodule update
# Configure, compile and install
autoreconf -i && ./configure && make && sudo make install
```

## 安装pecl扩展

```
#下载最新的amqp扩展
wget http://pecl.php.net/get/amqp-1.0.9.tgz
tar xvzf amqp-1.0.9.tgz
cd amqp-1.0.9 && phpize
./configure --with-amqp && make && sudo make install
```

记得在php.ini中加入amqp扩展：

```
extension=amqp.so
```

## 安装过程中可能会遇到的问题

1. 缺少libtool包

```
configure.ac: installing ./install-sh
configure.ac: installing ./missing
configure.ac:34: installing ./config.guess
configure.ac:34: installing ./config.sub
Makefile.am:3: Libtool library used but LIBTOOL is undefined
Makefile.am:3:
Makefile.am:3: The usual way to define LIBTOOL is to add AC_PROG_LIBTOOL
Makefile.am:3: to configure.ac and run aclocal and autoconf again.
Makefile.am: C objects in subdir but AM_PROG_CC_C_O not in configure.ac
Makefile.am: installing ./compile
Makefile.am: installing ./depcomp
autoreconf: automake failed with exit status: 1
```

解决办法，安装libtool，ubuntu：

```
sudo apt-get install libtool
```

其他系统类似

2. 如果还有其他问题，欢迎给我留言，我补上

## 使用

```
<?php
//配置信息
$conn_args = array(
    'host' => '127.0.0.1',
    'port' => '5672',
    'login' => 'guest',
    'password' => 'guest',
    'vhost'=>'/'
);
//创建连接
$conn = new AMQPConnection($conn_args);
if (!$conn->connect()) {
    die('Not connected :(' . PHP_EOL);
}
// Open Channel
$channel = new AMQPChannel($conn);

// Declare exchange
$exchange = new AMQPExchange($channel);
$exchange->setName('extest');
$exchange->setType('fanout');
$exchange->declare();

// Create Queue
$queue = new AMQPQueue($channel);
$queue->setName('qutest');
$queue->declare();

// Bind it on the exchange to routing.key
$exchange->bind('qutest', 'routing.key');

$data = array(
    'Name' => 'foobar',
    'Args'  => array("0", "1", "2", "3"),
);

//生产者，向RabbitMQ发送消息
$message = $exchange->publish(json_encode($data), 'key');
if (!$message) {
    echo 'Message not sent', PHP_EOL;
} else {
    echo 'Message sent!', PHP_EOL;
}

//消费者
while ($envelope = $queue->get(AMQP_AUTOACK)) {
    echo ($envelope->isRedelivery()) ? 'Redelivery' : 'New Message';
    echo PHP_EOL;
    echo $envelope->getBody(), PHP_EOL;
}
?>
```

更多请参考：<br />
[PHP官网](http://php.net/manual/en/amqp.examples.php)
