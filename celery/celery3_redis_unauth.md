启动靶场：
```
docker compose up -d
```

利用脚本：
```
python3 exploit.py localhost
```
查看结果：
```
docker compose logs celery
```
然后可以看到报错

检查容器内：
```
docker compose exec celery ls -l /tmp
```
可以看到成功创建了文件
