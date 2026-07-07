# 策略

策略是 Surge 处理网络流量的出站方式。策略在 `[Proxy]` 段落中声明。

## 内置策略

### DIRECT

将请求直接发送给目标主机，不经过任何代理。

```ini
DIRECT
```

### REJECT / REJECT-DROP / REJECT-NO-DROP {#reject-types}

拒绝请求。详见 [REJECT 策略](#reject-policy)。

### CELLULAR（仅限 iOS） {#cellular}

优先使用蜂窝网络而非 Wi-Fi。

### CELLULAR-ONLY（仅限 iOS） {#cellular-only}

仅使用蜂窝网络，蜂窝网络不可用时连接失败。

### HYBRID（仅限 iOS） {#hybrid}

尝试同时通过 Wi-Fi 和蜂窝网络建立连接。

### NO-HYBRID（仅限 iOS） {#no-hybrid}

如果 Wi-Fi 可用，则不尝试蜂窝网络。

### 别名 {#alias}

内置策略可直接使用，也可在 `[Proxy]` 段落中定义别名：

```ini
[Proxy]
On = direct
Off = reject
```

## 代理策略

### 代理类型 {#proxy-types}

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
| External | 外部代理程序（仅限 Mac），详见 [外部代理程序](#external-proxy-program) |

### 通用参数 {#proxy-params}

#### 代理链 (Proxy Chain) {#proxy-chain}

使用 `underlying-proxy` 参数指定一个代理作为底层传输：

```ini
ProxyVia = https, proxy.example.com, 443, username, password, underlying-proxy=ProxyA
```

#### TLS 通用参数 {#tls-params}

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

#### HTTP/HTTPS 专用参数 {#http-params}

```ini
# 始终使用 CONNECT 方法
ProxyHTTP = http, 1.2.3.4, 443, username, password, always-use-connect=true
```

#### SOCKS5 专用参数 {#socks5-params}

```ini
# 启用 UDP 转发
ProxySOCKS5 = socks5, 1.2.3.4, 443, username, password, udp-relay=true
```

#### Snell 参数 {#snell-params}

```ini
ProxySnell = snell, 1.2.3.4, 8000, psk=password, version=4, reuse=true, obfs=http, obfs-host=example.com, obfs-uri=/path
```

#### Shadowsocks 参数 {#shadowsocks-params}

```ini
ProxySS = ss, 1.2.3.4, 8000, encrypt-method=chacha20-ietf-poly1305, password=1234, udp-relay=true, obfs=tls, obfs-host=example.com

# 指定 UDP 端口
ProxySS = ss, 1.2.3.4, 8000, encrypt-method=chacha20-ietf-poly1305, password=1234, udp-relay=true, udp-port=8001
```

#### VMess 参数 {#vmess-params}

```ini
ProxyVMess = vmess, 1.2.3.4, 8000, username=uuid, ws=true, ws-path=/path, ws-headers=X-Header:value, tfo=true
```

- `ws`：可选，使用 WebSocket 传输层
- `ws-path`：可选
- `ws-headers`：可选
- `encrypt-method`：可选，`chacha20-ietf-poly1305` 或 `aes-128-gcm`
- `vmess-aead`：可选

#### Trojan 参数 {#trojan-params}

```ini
ProxyTrojan = trojan, 1.2.3.4, 443, password=password1, tfo=true
```

#### TUIC 参数 {#tuic-params}

```ini
ProxyTUIC = tuic, 1.2.3.4, 443, token=pwd, alpn=h3, udp-relay=true, skip-cert-verify=true
```

- `token`：必填
- `alpn`：可选，必须匹配服务器的 ALPN 设置
- `port-hopping`：可选，配置端口跳跃列表（如 `1234;5000-6000`）
- `port-hopping-interval`：可选，跳跃间隔，默认 30 秒

#### Hysteria 2 参数（iOS 5.8.0+ / Mac 5.4.0+） {#hysteria2-params}

```ini
ProxyHysteria = hysteria2, 1.2.3.4, 443, password=pwd, download-bandwidth=100, upload-bandwidth=50, udp-relay=true, skip-cert-verify=true, sni=example.com
```

- `download-bandwidth`：可选，Mbps
- `upload-bandwidth`：可选，Mbps
- `port-hopping`：可选，以分号分隔的端口或范围列表
- `port-hopping-interval`：可选，跳跃间隔，默认 30 秒

#### AnyTLS v2 参数（iOS 5.17.0+ / Mac 6.4.3+） {#anytls-params}

```ini
# iOS 5.17.0+ / Mac 6.4.3+
ProxyAnyTLS = anytls, 1.2.3.4, 443, password=pwd, skip-cert-verify=true, sni=example.com
```

- `reuse`：可选，默认启用连接复用，可设为 `false` 关闭

#### Shadow TLS {#shadow-tls}

Shadow TLS 是一种代理混淆器，可与任何基于 TCP 的代理一起使用。

```ini
[Proxy]
STLS-SNELL = snell, 1.2.3.4, 443, psk=pwd1, version=4, reuse=true, shadow-tls-password=pwd2, shadow-tls-version=3
```

参数：

- `shadow-tls-password`：必填
- `shadow-tls-sni`：可选，TLS 握手时发送的 SNI
- `shadow-tls-version`：可选，2 或 3，默认 2

### UDP 转发 {#udp-forward}

Surge 支持 SOCKS5、Snell v4/v5、Shadowsocks、Trojan、WireGuard、Hysteria 2 和 TUIC 协议的 UDP 转发。Shadowsocks 和 SOCKS5 需手动开启 `udp-relay=true`。

### 用于 TLS 代理的客户端证书 {#client-cert}

Surge 支持对基于 TLS 的代理进行客户端证书验证：

```ini
[Proxy]
Proxy = https, example.com, 443, client-cert=cert1

[Keystore]
cert1 = base64=<P12 的 Base64 字符串>, password=123456
```

### 测试 URL {#test-url-section}

```ini
# 为单个代理覆盖测试 URL
ProxyA = https, example.com, 443, username, password, test-url=http://www.gstatic.com/generate_204
```

## REJECT 策略 {#reject-policy}

### REJECT {#reject}

拒绝请求。HTTP 请求返回错误页面（可通过 `show-error-page-for-reject` 控制），TCP 连接直接断开。

### REJECT-DROP {#reject-drop}

拒绝请求。静默丢弃连接，不发送任何响应。适用于具有激进重试逻辑的应用，可避免请求风暴。

### REJECT-NO-DROP {#reject-no-drop}

与 REJECT 类似，但阻止 Surge 自动升级为 REJECT-DROP（默认策略：30 秒内 50 次匹配自动升级）。

### REJECT-TINYGIF（iOS 5.9.1+ / Mac 5.5.1+） {#reject-tinygif}

拒绝请求。HTTP 请求返回 1 像素透明 GIF 图片，用于广告拦截。

```ini
DOMAIN, example.com, REJECT-TINYGIF
```

### 预匹配拒绝 (Pre-matching Reject)（iOS 5.14.0+ / Mac 5.9.0+） {#pre-matching-reject}

在 DNS 解析和 TCP SYN 阶段以低开销快速拒绝请求，避免不必要的开销。

```ini
[Rule]
DOMAIN,ad.com,REJECT,pre-matching
```

标记为 `pre-matching` 的规则将在正常规则匹配之前生效，具有最高优先级。每 5 分钟只在最近请求列表中出现一次。

**支持的规则类型：**

- 域名类：`DOMAIN`、`DOMAIN-SUFFIX`、`DOMAIN-KEYWORD`、`DOMAIN-SET`、`DOMAIN-WILDCARD`
- IP 类：`IP-CIDR`、`IP-CIDR6`、`GEOIP`、`IP-ASN`
- 逻辑规则：`AND`、`OR`、`NOT`
- 其他：`SUBNET`、`DEST-PORT`、`SRC-PORT`、`SRC-IP`
- `RULE-SET` 也可使用，内容受上述限制

**预匹配行为差异：**

| 策略 | DNS 查询 | TCP SYN | UDP |
|------|----------|---------|-----|
| REJECT | 返回无记录响应 | 返回 RST | ICMP 禁止 |
| REJECT-DROP | 丢弃查询 | 丢弃 SYN 包 | 丢弃数据包 |
| REJECT-NO-DROP | 返回特殊 IP `198.18.0.244` | 返回 RST | ICMP 禁止 |

## WireGuard

WireGuard 可作为 Surge 的代理策略使用：

```ini
[Proxy]
WG = wireguard, interface-ip=10.0.0.2, interface-ipv6=fd00::2, dns=8.8.8.8, mtu=1280
WG = wireguard, peer=(public-key=<key>, allowed-ips=0.0.0.0/0, endpoint=example.com:51820, pre-shared-key=<key>, client-id=83/12/235)
```

### 参数说明 {#wg-params}

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

### 密码认证 {#ssh-password}

```ini
[Proxy]
proxy = ssh, 1.2.3.4, 22, username=root, password=pw
```

### 公钥认证 {#ssh-key}

```ini
[Proxy]
proxy = ssh, 1.2.3.4, 22, username=root, private-key=key1

[Keystore]
key1 = type=openssh-private-key, base64=[私钥文件Base64编码]
```

### 指纹验证 {#ssh-fingerprint}

```ini
proxy = ssh, 1.2.3.4, 22, username=root, password=pw, server-fingerprint="ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBk2No6..."
```

### 空闲超时 {#ssh-idle}

```ini
proxy = ssh, 1.2.3.4, 22, username=root, password=pw, idle-timeout=180
```

!!! note "SSH 限制"
    Surge 仅支持 `curve25519-sha256` 作为密钥交换算法，仅支持 `aes128-gcm` 作为加密算法。SSH 服务器需使用 OpenSSH v7.3 或更高版本。

## 外部代理程序（仅限 Mac） {#external-proxy-program}

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

### 出站参数 {#outbound-params}

以下参数对内置策略和代理策略均可用。

#### `interface` {#interface}

强制使用指定的出站网络接口：

```ini
ProxyHTTP = http, 1.2.3.4, 443, username, password, interface=en2
Corp-VPN = direct, interface=utun0
```

#### `allow-other-interface` {#allow-other-interface}

当所需接口不可用时，允许使用默认接口（默认 false）：

```ini
ProxyHTTP = http, 1.2.3.4, 443, username, password, interface=en2, allow-other-interface=true
```

#### `dns-follow-interface`（iOS 5.15.2+ / Mac 5.2.0+） {#dns-follow-interface}

让 `interface` 参数对 DNS 查询也生效。

#### `no-error-alert` {#no-error-alert}

不显示该策略的错误警告。

#### `ip-version` {#ip-version}

选择 IPv4/IPv6 协议行为（仅对代理服务器连接有效）：

- `dual`：默认，使用最快链路
- `v4-only`、`v6-only`
- `prefer-v4`、`prefer-v6`

#### `hybrid`（仅限 iOS） {#hybrid-param}

同时建立蜂窝数据和 Wi-Fi 连接，使用较快链路。

#### `tfo` {#tfo}

启用 TCP Fast Open：

```ini
ProxyA = https, example.com, 443, username, password, tfo=true
```

#### `tos` {#tos}

自定义 IP TOS 值（十进制或十六进制，默认 0）。

#### `ecn`（iOS 5.8.0+ / Mac 5.4.0+） {#ecn}

启用显式拥塞通知，高丢包环境下可提升性能。

#### `block-quic`（iOS 5.8.0+ / Mac 5.4.0+） {#block-quic}

阻断 QUIC 流量，使客户端回退到 HTTPS/TCP：

- `auto`：自动判断
- `on`：阻断
- `off`：不阻断

### 测试参数 {#test-params}

#### `test-url` {#test-url-param}

覆盖默认的连通性测试 URL：

```ini
ProxyA = https, example.com, 443, username, password, test-url=http://www.gstatic.com/generate_204
```

#### `test-timeout` {#test-timeout}

覆盖全局测试超时时间（秒）。

#### `test-udp` {#test-udp}

通过 DNS 查询测试 UDP 中继：

```ini
test-udp=google.com@1.1.1.1
```
