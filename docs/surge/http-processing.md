# HTTP 处理

## HTTPS 解密 (MitM) {#mitm}

Surge 可以通过中间人攻击 (MitM) 解密 HTTPS 流量。证书生成器内置于 Surge Dashboard（Mac）和 Surge iOS 配置编辑器中，证书在本地生成。

### 基本配置

```ini
[MITM]
ca-p12 = MIIJtQ.........         # CA 证书的 Base64 编码
ca-passphrase = password          # 证书密码
hostname = *                       # 需要解密的主机名
h2 = true                          # 启用 HTTP/2 MITM
```

### hostname 参数

`hostname` 为 Host List 类型，指定需要解密的主机名：

```ini
# 排除 Apple 网站，解密其他所有流量
hostname = -*.apple.com, -*.icloud.com, *

# 仅解密指定域名
hostname = *google.com, *youtube.com
```

### 选项

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `skip-server-cert-verify` | 布尔 | false | 不验证远程主机证书 |
| `h2` | 布尔 | false | 通过 HTTP/2 执行 MITM |
| `client-source-address` | 列表 | - | 按客户端 IP 启用 MITM |
| `auto-quic-block` | 布尔 | true | 自动阻断 QUIC 连接（iOS 5.8.0+ / Mac 5.4.0+） |

`client-source-address` 示例：

```ini
[MITM]
client-source-address = -192.168.1.2, 0.0.0.0/0
hostname = *
```

!!! warning "固定证书"
    某些应用使用固定证书 (Certificate Pinning)，对这类主机启用 MITM 可能导致连接问题。

## URL 重写 (URL Rewrite) {#url-rewrite}

重写请求的 URL 或根据 URL 拒绝请求。支持三种模式：

### Header 模式

修改请求头并将请求重定向到新主机。客户端无感知：

```ini
[URL Rewrite]
^http://www\.google\.cn http://www.google.com header
```

### 302 模式

直接返回 302 重定向响应。HTTPS 请求需启用 MitM：

```ini
[URL Rewrite]
^http://yachen\.com https://yach.me 302
```

### Reject 模式

匹配时拒绝请求。HTTPS 请求需启用 MitM：

```ini
[URL Rewrite]
^http://ad\.com/ad\.png _ reject
```

## 请求头重写 (Header Rewrite) {#header-rewrite}

重写请求头或响应头：

```ini
[Header Rewrite]
http-request ^http://example.com header-add DNT 1           # 添加请求头
http-request ^http://example.com header-del Cookie           # 删除请求头
http-request ^http://example.com header-replace User-Agent Unknown  # 替换值
http-response ^http://example.com header-replace-regex Date 2022 2023  # 正则替换
```

### 语法

```
[HTTP走向] [URL正则] [动作] [字段名] [值]
```

- HTTP 走向：`http-request` 或 `http-response`（省略时默认为 `http-request`）
- 动作类型：`header-add`、`header-del`、`header-replace`、`header-replace-regex`

### header-add 示例

```ini
[Header Rewrite]
http-request ^http://example.com header-add DNT 1

# 修改前:
# GET /index.html HTTP/1.1
# Host: example.com
# User-Agent: Safari/603.1.30

# 修改后:
# GET /index.html HTTP/1.1
# Host: example.com
# User-Agent: Safari/603.1.30
# DNT: 1
```

### header-replace-regex 示例

```ini
[Header Rewrite]
http-request ^http://example.com header-replace-regex User-Agent Safari Chrome

# User-Agent 中的 "Safari" 被替换为 "Chrome"
```

### 组合使用

```ini
[Header Rewrite]
^http://example.com header-del DNT
^http://example.com header-add DNT 1
```

## 请求体重写 (Body Rewrite)（iOS 5.10.0+ / Mac 5.6.0+） {#body-rewrite}

使用正则表达式替换 HTTP 请求或响应的请求体内容：

```ini
[Body Rewrite]
http-request ^http(s)?://example\.com value abc
http-response ^http(s)?://example\.com documents Surge
```

### 语法

```
http-request [URL正则] [搜索正则] [替换内容]
http-response [URL正则] [搜索正则] [替换内容]
```

支持连续替换：

```ini
http-response ^https?://example\.com/ regex1 replacement1 regex2 replacement2
```

### JQ 请求体重写（iOS 5.14.0+ / Mac 5.9.0+）

使用 JQ 表达式操作 JSON 请求体：

```ini
http-response-jq ^http://httpbingo.org/anything '.headers |= with_entries(select(.key | test("^X-") | not))'
```

## 模拟响应 (Mock / Map Local) {#mock}

模拟 HTTP 服务器并返回静态响应：

```ini
[Map Local]
^http://surgetest\.com/json data-type=text data="{}" status-code=500
^http://surgetest\.com/gif data-type=tiny-gif status-code=200
^http://surgetest\.com/file data-type=file data="data/map-local.json" header="a:b|foo:bar"
^http://surgetest\.com/base64 data="dGVzdA==" data-type=base64
```

### data-type 参数

| 类型 | 说明 |
|------|------|
| `file` | 返回文件内容，路径相对于配置文件目录 |
| `text` | 返回 UTF-8 文本 |
| `tiny-gif` | 返回 1 像素 GIF |
| `base64` | 返回 Base64 编码的二进制数据 |

### 其他参数

- `status-code`：自定义 HTTP 状态码
- `header`：自定义响应头，使用 `|` 分隔多个键值对
- `data`：数据内容

### Content-Type

Surge 会尽可能自动补全 Content-Type：

- `file`：根据扩展名映射 MIME，失败则使用 `application/octet-stream`
- `text`：默认 `plain/text`
- `tiny-gif`：默认 `image/gif`
- `base64`：默认 `application/octet-stream`
