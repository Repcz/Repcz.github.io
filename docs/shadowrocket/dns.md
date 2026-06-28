# DNS 设置

> 本文档基于 [LOWERTOP/Shadowrocket](https://github.com/LOWERTOP/Shadowrocket/blob/main/README.md) 整理

## DNS 覆写

`配置 ⓘ > 通用 > DNS 覆写`

DNS 覆写仅针对**直连类域名**进行解析，代理类域名经由代理服务器解析。

支持并发查询多个 DNS，使用最快返回的结果。`system` 表示使用系统 DNS。

### 普通 DNS

```ini
dns-server = 223.5.5.5,119.29.29.29
```

### 加密 DNS

**DNS-over-HTTPS (DoH)**

```ini
dns-server = https://dns.alidns.com/dns-query
```

DoH 支持 HTTP/3，可在链接后加 `#no-h3` 关闭。强制使用 H3 的写法：

```ini
dns-server = h3://dns.alidns.com/dns-query
```

**DNS-over-QUIC (DoQ)**

```ini
dns-server = quic://223.5.5.5
```

**DNS-over-TLS (DoT)**

```ini
dns-server = tls://223.5.5.5
```

### DNS-over-PROXY

通过代理转发 DNS 查询请求，适用于需要加密 DNS 防污染的场景。

**使用默认节点转发**

```ini
dns-server=https://dns.google/dns-query#proxy
```

或显式指定：

```ini
dns-server=https://dns.google/dns-query#proxy=proxy
```

**使用指定节点转发**

节点名称需 URL 编码：

```ini
# 使用名为"香港 01"的节点
dns-server=https://dns.google/dns-query#proxy=%E9%A6%99%E6%B8%AF%2001
```

**EDNS Client Subnet (ECS)**

向 DNS 服务器传递客户端子网信息，返回最优结果：

```ini
dns-server=https://dns.google/dns-query#ecs=120.76.0.0/14|2620:149:af0::10/56&ecs-override=true
```

**ECS 强制覆盖**

```ini
dns-server=https://dns.google/dns-query#proxy=name&ecs=1.1.0.0/14|2620:149:af0::10/56&ecs-override=true
```

---

## 备用 DNS

```ini
fallback-dns-server = system
```

- 当主 DNS 查询失败或超时 2 秒后回退
- 多个 DNS 用逗号分隔
- 留空或 `system` 表示回退到系统 DNS

---

## no-resolve 的作用

域名请求遇到 IP 类规则时，Shadowrocket 会向本地 DNS 发送查询以判断主机 IP 是否匹配规则。

- **不加 `no-resolve`**：触发本地 DNS 查询
- **加 `no-resolve`**：跳过此规则，不触发本地 DNS 查询

```ini
IP-CIDR,172.16.0.0/12,DIRECT,no-resolve
```

在规则集中使用 `no-resolve` 可避免不必要的 DNS 查询，提升性能。

---

## DNS 劫持

```ini
hijack-dns = 8.8.8.8
```

某些设备或应用使用硬编码 DNS（如 Netflix 使用 Google DNS `8.8.8.8`），可用此选项劫持查询。

---

## 相关隐式参数

以下参数需在配置文件纯文本 `[General]` 中设置，参见[隐式参数](config.md#隐式参数)：

| 参数 | 说明 |
|------|------|
| `dns-direct-system` | 直连域名使用系统 DNS 查询 |
| `dns-direct-fallback-proxy` | 直连域名解析失败后使用代理 |
| `proxy-dns-server` | 指定 DNS 解析所有节点域名 |
| `always-ip-address` | 强制所有域名使用本地 DNS 解析 |
| `allow-dns-svcb` | 允许 DNS SVCB 查询 |
| `allow-dns-all` | 允许所有 DNS 查询类型正常转发 |
