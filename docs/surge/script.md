# 脚本

Surge 支持使用 JavaScript 扩展能力。脚本功能需要 Surge iOS 4 或 Surge Mac 3.3.0+。

## 脚本段落 {#script-section}

```ini
[Script]
# HTTP 响应脚本
script1 = type=http-response, pattern=^http://www.example.com/test, script-path=test.js, max-size=16384, debug=true

# 计划任务
script2 = type=cron, cronexp="* * * * *", script-path=fired.js

# HTTP 请求脚本
script3 = type=http-request, pattern=^http://httpbin.org, script-path=http-request.js, requires-body=true

# DNS 脚本
script4 = type=dns, script-path=dns.js, debug=true

# 规则脚本
script5 = type=rule, script-path=rule.js

# 通用脚本（面板）
script6 = type=generic, script-path=panel.js
```

### 公共参数 {#common-params}

| 参数 | 说明 |
|------|------|
| `type` | 脚本类型：`http-request`、`http-response`、`cron`、`event`、`dns`、`rule`、`generic` |
| `script-path` | 脚本路径（相对路径、绝对路径或 URL） |
| `script-update-interval` | URL 脚本的更新间隔（秒） |
| `debug` | 开启调试模式 |
| `timeout` | 脚本最长运行时间（默认 5 秒） |
| `argument` | 通过 `$argument` 传递给脚本 |
| `engine` | JavaScript 引擎选择 |

### HTTP 脚本专用参数 {#http-script-params}

| 参数 | 说明 |
|------|------|
| `pattern` | 匹配 URL 的正则表达式 |
| `requires-body` | 允许修改请求/响应体（默认 false，开销大） |
| `max-size` | 请求/响应体的最大大小（默认 131072 字节/128KB） |
| `binary-body-mode` | 原始二进制数据以 `Uint8Array` 形式传递 |

## 基本限制 {#basic-limits}

- 脚本支持异步操作
- 必须调用 `$done(value)` 表示完成
- 默认超时 5 秒

## 公共 API {#public-api}

### 网络信息 {#network-info}

```javascript
$network          // 网络环境详细信息
$network.wifi.ssid // 当前 Wi-Fi SSID
$network.dns       // DNS 服务器列表
```

### 脚本信息 {#script-info}

```javascript
$script.name          // 脚本名称
$script.startTime     // 脚本开始时间
$script.type          // 脚本类型
```

### 环境信息 {#environment-info}

```javascript
$environment.system          // "iOS" 或 "macOS"
$environment.surge-build     // Surge 构建号
$environment.surge-version   // Surge 版本
$environment.language        // Surge UI 语言
$environment.device-model    // 设备型号
```

### 持久化存储 {#persistent-store}

```javascript
// 写入数据
$persistentStore.write(data, [key])

// 读取数据
$persistentStore.read([key])
```

相同 `script-path` 的脚本共享存储池。使用 `key` 可在不同脚本间共享数据。

!!! tip "Mac 持久化存储路径"
    `~/Library/Application Support/com.nssurge.surge-mac/SGJSVMPersistentStore/`

### 控制 Surge {#control-surge}

```javascript
$httpAPI(method, path, body, callback)
```

无需鉴权参数，可调用所有 HTTP API 控制 Surge。

### HTTP 客户端 {#http-client}

```javascript
$httpClient.get(url, callback)
$httpClient.post(url, callback)
$httpClient.put(url, callback)
$httpClient.delete(url, callback)
$httpClient.head(url, callback)
$httpClient.options(url, callback)
$httpClient.patch(url, callback)
```

参数可以是 URL 字符串或选项对象：

```javascript
$httpClient.post({
  url: "http://www.example.com/",
  headers: { "Content-Type": "application/json" },
  body: "{}",
  timeout: 5
}, function(error, response, data) {
  if (error) {
    console.log("请求失败");
  } else {
    console.log(response.status, response.headers);
  }
});
```

### 通知 {#notification}

```javascript
// 推送通知
$notification.post(title, subtitle, content)
// 带 URL 的通知
$notification.post(title, subtitle, content, url)
```

### 其他工具 {#other-utils}

```javascript
// 策略组控制
$surge.setSelectGroupPolicy(groupName, policyName)

// 日志
console.log(message)
```

## HTTP 请求脚本 (http-request) {#http-request}

在请求发送到服务器之前执行：

```javascript
let headers = $request.headers;
headers['X-Modified-By'] = 'Surge';

$done({headers});
```

### 输入参数 {#http-request-input}

- `$request.url` — 请求 URL
- `$request.method` — HTTP 方法
- `$request.id` — 唯一 ID
- `$request.headers` — 请求头
- `$request.body` — 请求体（需 `requires-body=true`）

### 返回参数 {#http-request-return}

- `url` — 新的 URL（不会自动更新 Host 请求头）
- `headers` — 新的请求头
- `body` — 新的请求体（需 `requires-body=true`）
- `response` — 直接返回 HTTP 响应而无需真实网络请求，包含 `status`、`headers`、`body`

调用 `$done();` 中止请求，`$done({});` 保持请求不变。

示例 — 直接返回响应：

```javascript
$done({
  response: {
    status: 200,
    headers: { 'Content-Type': 'application/json' },
    body: '{"message": "OK"}'
  }
});
```

## HTTP 响应脚本 (http-response) {#http-response}

在收到服务器响应后执行：

```javascript
let headers = $response.headers;
headers['X-Modified-By'] = 'Surge';

$done({headers});
```

### 输入参数 {#http-response-input}

- `$request.url`、`$request.method`、`$request.id`、`$request.headers`
- `$response.status` — HTTP 状态码
- `$response.headers` — 响应头
- `$response.body` — 响应体（需 `requires-body=true`）

### 返回参数 {#http-response-return}

- `body` — 新的响应体
- `headers` — 新的响应头
- `status` — 新的状态码

调用 `$done();` 中止请求，`$done({});` 保持响应不变。

## 规则脚本 (rule) {#rule-script}

使用脚本作为规则：

```ini
[Script]
ssid-rule = type=rule, script-path=ssid-rule.js

[Rule]
SCRIPT, ssid-rule, DIRECT
```

```javascript
var hostnameMatched = ($request.hostname === 'home.com');
var ssidMatched = ($network.wifi.ssid === 'My Home');

$done({matched: (hostnameMatched && ssidMatched)});
```

### 输入参数 {#rule-script-input}

- `$request.hostname`、`$request.destPort`、`$request.processPath`
- `$request.userAgent`、`$request.url`、`$request.sourceIP`
- `$request.listenPort`、`$request.dnsResult`、`$request.srcPort`
- `$request.protocol`

使用 `requires-resolve` 选项触发 DNS 查询：

```ini
SCRIPT, ssid-rule, DIRECT, requires-resolve
```

## 事件脚本 (event) {#event-script}

在指定事件发生时执行脚本：

```ini
[Script]
network-changed = type=event, event-name=network-changed, script-path=network-changed.js
```

### 事件类型 {#event-types}

**network-changed** — 系统网络变化时触发：

```javascript
$notification.post('DNS Update', $network.dns.join(', '));
$done();
```

**notification** — Surge 显示通知时触发：

```javascript
console.log($event.data);
$done();
```

## DNS 脚本 (dns) {#dns-script}

使用脚本作为 DNS 解析器：

```ini
[Script]
dnspod = type=dns, script-path=dnspod.js

[Host]
example.com = script:dnspod
*.example.com = script:dnspod
```

```javascript
$httpClient.get('http://119.29.29.29/d?dn=' + $domain, function(error, response, data) {
  if (error) {
    $done({}); // 回退到标准 DNS 查询
  } else {
    $done({addresses: data.split(';'), ttl: 600});
  }
});
```

### 返回值 {#dns-return-values}

| 返回值 | 说明 |
|--------|------|
| `address<String>` | 单个 IP 地址 |
| `addresses<Array>` | 多个 IP 地址 |
| `server<String>` | 通过指定 DNS 服务器解析 |
| `servers<Array>` | 通过多个 DNS 服务器解析 |
| `ttl<Number>` | 缓存时间（秒） |

## 计划任务脚本 (cron) {#cron-script}

在指定时间执行脚本：

```ini
[Script]
cron = type=cron, cronexp="0 2 * * *", script-path=cron.js
```

```javascript
$surge.setSelectGroupPolicy('Group', 'Proxy');
$done();
```

### cron 表达式示例 {#cron-examples}

| 表达式 | 说明 |
|--------|------|
| `0 2 * * *` | 每天凌晨 2 点 |
| `0 5,17 * * *` | 每天早上 5 点和下午 5 点 |
| `* * * * *` | 每分钟 |
| `* * * * * *` | 每秒 |
| `0 17 * * sun` | 每个星期日下午 5 点 |
| `*/10 * * * *` | 每 10 分钟 |
