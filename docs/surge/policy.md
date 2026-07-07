# 策略

策略是 Surge 处理网络流量的出站方式。策略在 `[Proxy]` 段落中声明。

## 内置策略

### DIRECT

将请求直接发送给目标主机，不经过任何代理。

```ini
DIRECT
```

### REJECT / REJECT-DROP / REJECT-NO-DROP

拒绝请求。详见 [REJECT 策略](#reject-策略)。

### CELLULAR（仅限 iOS）

优先使用蜂窝网络而非 Wi-Fi。

### CELLULAR-ONLY（仅限 iOS）

仅使用蜂窝网络，蜂窝网络不可用时连接失败。

### HYBRID（仅限 iOS）

尝试同时通过 Wi-Fi 和蜂窝网络建立连接。

### NO-HYBRID（仅限 iOS）

如果 Wi-Fi 可用，则不尝试蜂窝网络。

### 别名

内置策略可直接使用，也可在 `[Proxy]` 段落中定义别名：

```ini
[Proxy]
On = direct
Off = reject
```

## 代理策略

### 代理类型

Surge 支持以下代理协议：

| 类型 | 语法示例 |
|------|---------|
| HTTP | `ProxyHTTP = http, 1.2.3.4, 443, username, password` |
| HTTPS | `ProxyHTTPS = https, 1.2.3.4, 443, username, password` |
| SOCKS5 | `ProxySOCKS5 = socks5, 1.2.3.4, 443, username, password` |
| SOCKS5 over TLS | `ProxySOCKS5TLS = socks5-tls, 1.2.3.4, 443, username, password` |
| Snell | `ProxySnell = snell, 1.2.3.4, 8000, psk=password, version=4` |
| Shadowsocks | `ProxySS = ss, 1.2.3.4, 8000, encrypt-method=chacha20-ietf-poly1305, password=1234` |
| VMess | `ProxyVMess = vmess, 1.2.3.4, 8000, username=uuid` |
| Trojan | `ProxyTrojan = trojan, 1.2.3.4, 443, password=password1` |
| TUIC | `ProxyTUIC = tuic, 1.2.3.4, 443, token=pwd, alpn=h3` |
| Hysteria 2 | `ProxyHysteria = hysteria2, 1.2.3.4, 443, password=pwd, download-bandwidth=100` |
| AnyTLS | `ProxyAnyTLS = anytls, 1.2.3.4, 443, password=pwd`（iOS 5.17.0+ / Mac 6.4.3+） |
| WireGuard | 作为 L3 VPN 使用，详见 [WireGuard 策略](#wireguard) |
| SSH | 相当于 `ssh -D`，详见 [SSH 策略](#ssh) |
| External | 外部代理程序（仅限 Mac），详见 [外部代理程序](#外部代理程序仅限-mac) |

### 通用参数

#### 代理链 (Proxy Chain)

使用 `underlying-proxy` 参数指定一个代理作为底层传输：

```ini
ProxyVia = https, proxy.example.com, 443, username, password, underlying-proxy=ProxyA
```

#### TLS 通用参数

```ini
# 跳过证书验证
ProxyHTTPS = https, 1.2.3.4, 443, username, password, skip-cert-verify=true

# 自定义 SNI
ProxyHTTPS = https, 1.2.3.4, 443, username, password, sni=example.com

# 关闭 SNI
ProxyHTTPS = https, 1.2.3.4, 443, username, password, sni=off

# 固定证书指纹
ProxyHTTPS = https, 1.2.3.4, 443, skip-cert-verify=false, server-cert-fingerprint-sha256=xxxx
```

#### HTTP/HTTPS 专用参数

```ini
# 始终使用 CONNECT 方法
ProxyHTTP = http, 1.2.3.4, 443, username, password, always-use-connect=true
```

#### SOCKS5 专用参数

```ini
# 启用 UDP 转发
ProxySOCKS5 = socks5, 1.2.3.4, 443, username, password, udp-relay=true
```

#### Snell 参数

```ini
ProxySnell = snell, 1.2.3.4, 8000, psk=password, version=4, reuse=true, obfs=http, obfs-host=example.com, obfs-uri=/path
```

#### Shadowsocks 参数

```ini
ProxySS = ss, 1.2.3.4, 8000, encrypt-method=chacha20-ietf-poly1305, password=1234, udp-relay=true, obfs=tls, obfs-host=example.com

# 指定 UDP 端口
ProxySS = ss, 1.2.3.4, 8000, encrypt-method=chacha20-ietf-poly1305, password=1234, udp-relay=true, udp-port=8001
```

#### VMess 参数

```ini
ProxyVMess = vmess, 1.2.3.4, 8000, username=uuid, ws=true, ws-path=/path, ws-headers=X-Header:value, tfo=true
```

#### Trojan 参数

```ini
ProxyTrojan = trojan, 1.2.3.4, 443, password=password1, tfo=true
```

#### TUIC 参数

```ini
ProxyTUIC = tuic, 1.2.3.4, 443, token=pwd, alpn=h3, udp-relay=true, skip-cert-verify=true
```

#### Hysteria 2 参数

```ini
ProxyHysteria = hysteria2, 1.2.3.4, 443, password=pwd, download-bandwidth=100, upload-bandwidth=50, udp-relay=true, skip-cert-verify=true, sni=example.com
```

#### AnyTLS 参数

```ini
# iOS 5.17.0+ / Mac 6.4.3+
ProxyAnyTLS = anytls, 1.2.3.4, 443, password=pwd, skip-cert-verify=true, sni=example.com
```

### UDP 转发

Surge 支持 SOCKS5、Snell v4/v5、Shadowsocks、Trojan、WireGuard、Hysteria 2 和 TUIC 协议的 UDP 转发。Shadowsocks 和 SOCKS5 需手动开启 `udp-relay=true`。

### 测试 URL

```ini
# 为单个代理覆盖测试 URL
ProxyA = https, example.com, 443, username, password, test-url=http://www.gstatic.com/generate_204
```

## REJECT 策略

### REJECT

返回拒绝响应（HTTP 请求返回错误页面，TCP 连接直接断开）。

### REJECT-DROP

静默丢弃数据包，不发送任何响应。适用于 UDP 协议。

### REJECT-NO-DROP

与 REJECT 类似，但不丢弃数据包（TCP 连接会发送 RST）。

### REJECT-TINYGIF（iOS 5.9.1+ / Mac 5.5.1+）

针对 HTTP 请求返回 1 像素透明 GIF 图片。

```ini
DOMAIN, example.com, REJECT-TINYGIF
```

## WireGuard

WireGuard 可作为 Surge 的代理策略使用：

```ini
[Proxy]
WG = wireguard, interface-ip=10.0.0.2, interface-ipv6=fd00::2, dns=8.8.8.8, mtu=1280
WG = wireguard, peer=(public-key=<key>, allowed-ips=0.0.0.0/0, endpoint=example.com:51820, pre-shared-key=<key>, client-id=83/12/235)
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `interface-ip` | WireGuard 接口的 IPv4 地址 |
| `interface-ipv6` | WireGuard 接口的 IPv6 地址 |
| `dns` | DNS 服务器 |
| `mtu` | MTU 值（默认 1280） |
| `public-key` | 对端公钥 |
| `allowed-ips` | 允许的 IP 范围 |
| `endpoint` | 对端地址和端口 |
| `pre-shared-key` | 预共享密钥 |
| `keepalive` | PersistentKeepalive 间隔（秒） |
| `self-ip` | 自用 IP（绕过 Surge VIF） |
| `self-ipv6` | 自用 IPv6 |
| `client-id` | 自定义保留位（如 Cloudflare WARP 使用） |
| `ecn` | ECN 支持（iOS 5.8.0+ / Mac 5.4.0+） |

## SSH

Surge 支持 SSH 作为代理策略，相当于 `ssh -D`。

### 密码认证

```ini
[Proxy]
proxy = ssh, 1.2.3.4, 22, username=root, password=pw
```

### 公钥认证

```ini
[Proxy]
proxy = ssh, 1.2.3.4, 22, username=root, private-key=key1

[Keystore]
key1 = type=openssh-private-key, base64=[私钥文件Base64编码]
```

### 指纹验证

```ini
proxy = ssh, 1.2.3.4, 22, username=root, password=pw, server-fingerprint="ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBk2No6..."
```

### 空闲超时

```ini
proxy = ssh, 1.2.3.4, 22, username=root, password=pw, idle-timeout=180
```

!!! note "SSH 限制"
    Surge 仅支持 `curve25519-sha256` 作为密钥交换算法，仅支持 `aes128-gcm` 作为加密算法。SSH 服务器需使用 OpenSSH v7.3 或更高版本。

## 外部代理程序（仅限 Mac）

Surge Mac 支持运行外部程序作为代理策略：

```ini
[Proxy]
external = external, exec="/usr/bin/ssh", args="-D", args="127.0.0.1:1080", args="11.22.33.44", local-port=1080, addresses=11.22.33.44
```

- `exec` 和 `local-port` 必填，`args` 和 `addresses` 可选
- Surge 将请求转发到 SOCKS5 `127.0.0.1:[local-port]`
- 外部进程崩溃后自动重启
- 增强模式下自动排除 `addresses` 中的 IP
- 日志输出到 `/tmp/Surge-External-xxxxxx.log`

## 通用策略参数

### `test-url`

覆盖默认的连通性测试 URL：

```ini
ProxyA = https, example.com, 443, username, password, test-url=http://www.gstatic.com/generate_204
```

### `tfo` (TCP Fast Open)

```ini
ProxyA = https, example.com, 443, username, password, tfo=true
```

### `no-alert`

```ini
ProxyA = https, example.com, 443, username, password, no-alert=true
```
