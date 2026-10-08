开启环境：
```
docker compose up -d
```
访问页面：
```
http://localhost/uploadfiles/nginx.png
```
会显示一张图片
但是访问：
```
http://localhost/uploadfiles/nginx.png/.php
```
往下拉就会显示phpinfo
因为添加了`/.php`被解析成php文件

经过我的测试，jpg文件解析不了，不知道，文档说可以
且需要在.png文件里加段代码：
```
<?phpinfo?>
```
即可


