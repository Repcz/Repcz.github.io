# DNS

Surge 使用自定义的 DNS 客户端实现，行为可能与操作系统的 DNS 客户端不同。

## 上游 DNS 服务器

### 基本配置

```ini
[General]
dns-server = 8.8.8.8, 8.8.4.4
```

使用 `system` 关键字追加系统 DNS：

```ini
[General]
dns-server = system, 8.8.8.8, 1.1.1.1
```

### 技术细节

- Surge 同时向所有 DNS 服务器发起查询（类似 dnsmasq 的 `--all-servers` 模式），使用最先返回的响应
- 如果 2 秒内无响应，再次查询所有服务器
- 重试四次后仍无响应则报告 DNS 错误
- 当 IPv6 启用时，同时发送 A 和 AAAA 请求

## 本地 DNS 映射

`[Host]` 段落用于本地 DNS 映射，功能比 `/etc/hosts` 更强大。

### 静态映射

```ini
[Host]
abc.com = 1.2.3.4
```

### 通配符

使用 `*` 前缀通配所有子域名：

```ini
[Host]
*.dev = 6.7.8.9
```

!!! warning "通配符匹配规则"
    Surge 使用简单的字符串匹配。`*google.com` 会匹配 `google.com`、`foo.google.com` 以及 `bargoogle.com`。而 `*.google.com` **不会**匹配 `google.com`。

### 别名 (CNAME)

```ini
[Host]
foo.com = bar.com
```

### 分配 DNS 服务器

为指定域名分配特定的 DNS 服务器：

```ini
[Host]
bar.com = server:8.8.8.8
bar.com = server:https://cloudflare-dns.com/dns-query
bar.com = server:system
bar.com = server:syslib    # 保留在 Surge 内部但转发给系统 DNS
```

默认情况下，所有 `.local` 后缀的主机名由系统解析。

### 引用规则集（Mac 5.10.0+）

将 `DOMAIN-SET` 或 `RULE-SET` 绑定到 DNS 映射条目：

```ini
[Host]
DOMAIN-SET:https://example.com/domains.txt = server:https://doh.example.com/dns-query
RULE-SET:https://example.com/rules.txt = 10.0.0.10
```

### 代理请求使用本地 DNS

```ini
[General]
use-local-host-item-for-proxy=true
```

启用后，对于符合本地 DNS 映射的请求，Surge 使用本地 IP 地址而非原始域名发送代理请求。仅对使用 IP 地址的记录有效。

## 加密 DNS

### 支持的协议

| 协议 | 格式 |
|------|------|
| DNS over HTTPS (DoH) | `https://example.com/dns-query` |
| DNS over HTTP/3 (DoH3) | `h3://example.com/dns-query` |
| DNS over QUIC (DoQ) | `quic://example.com` |

### 全局加密 DNS

```ini
[General]
encrypted-dns-server = https://8.8.8.8/dns-query
encrypted-dns-server = https://1.1.1.1/dns-query, https://8.8.8.8/dns-query
```

### 指定域名加密 DNS

```ini
[Host]
example.com = server:https://cloudflare-dns.com/dns-query
```

### 通过代理使用加密 DNS

```ini
[General]
encrypted-dns-follow-outbound-mode=true
```

启用后，所有加密 DNS 连接遵循出站模式设置。可以为 DoH 主机名配置规则以使用代理：

```ini
[Rule]
PROTOCOL, DOH, Proxy    # 匹配 Surge 自身发出的 DoH 请求
PROTOCOL, DOH3, Proxy   # 匹配 DoH3 请求
PROTOCOL, DOQ, Proxy    # 匹配 DoQ 请求
```

## 虚假 IP (Fake IP)

Surge 在 VIF 模式下使用 `198.18.0.0/15` 地址段作为虚假 IP 地址。DNS 查询会被 Surge 的 DNS 客户端截获并返回虚假 IP，请求实际由 Surge 代理引擎处理。

### 相关设置

```ini
[General]
# 劫持所有发往 53 端口的 DNS 查询
hijack-dns = *:53

# 当 Surge VIF 处理 DNS 查询时返回真实 IP 地址
always-real-ip = *.local
```

### hijack-dns

默认仅对发送至 Surge DNS 地址 (`198.18.0.2`) 的 DNS 查询返回虚假 IP。使用 `hijack-dns` 劫持发送至标准 DNS 的查询：

```ini
hijack-dns = *:53      # 劫持所有 DNS 查询
hijack-dns = 8.8.8.8:53  # 劫持特定 DNS 服务器
```

虚假 DNS 响应器监听地址：
- IPv4: `198.18.0.2`
- IPv6: `fd00:6152::2`
