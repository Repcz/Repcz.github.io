# 规则系统

Surge 的规则系统用于决定网络流量的出站策略。规则按顺序逐条匹配，命中的第一条规则生效。

## 规则语法

```ini
[Rule]
# 规则类型, 参数, 策略
DOMAIN-SUFFIX, google.com, Proxy
GEOIP, CN, DIRECT
FINAL, Proxy
```

## 基于域名的规则 {#domain-based-rule}

### DOMAIN

精确匹配域名：

```ini
DOMAIN, example.com, DIRECT
```

### DOMAIN-SUFFIX

匹配域名后缀（包括完整域名）：

```ini
DOMAIN-SUFFIX, google.com, Proxy
```
可匹配 `google.com`、`www.google.com`、`mail.google.com`。

### DOMAIN-KEYWORD

匹配域名中包含的关键字：

```ini
DOMAIN-KEYWORD, google, Proxy
```
可匹配 `google.com`、`www.google.com.hk`。

### DOMAIN-SET

专为大量域名设计，支持数千条记录的快速搜索。文件中的每一行都是一个域名，以 `.` 开头则匹配所有子域名及该域名本身：

```ini
DOMAIN-SET, https://example.com/domains.txt, Proxy
```

支持 `extended-matching` 参数，同时匹配 SNI 和 HTTP Host 请求头（iOS 5.8.0+ / Mac 5.4.0+）：

```ini
DOMAIN-SET, https://example.com/domains.txt, Proxy, extended-matching
```

## 基于 IP 的规则 {#ip-based-rule}

### IP-CIDR

匹配 IPv4 地址范围：

```ini
IP-CIDR, 192.168.0.0/16, DIRECT
IP-CIDR, 10.0.0.0/8, DIRECT
IP-CIDR, 172.16.0.0/12, DIRECT
```

从 Surge Mac 6.0.0 开始，支持不带 `/` 掩码的单一 IPv4 地址（视为 `/32`）。

### IP-CIDR6

匹配 IPv6 地址范围：

```ini
IP-CIDR6, 2001:db8:abcd:8000::/50, DIRECT
```

### GEOIP

根据 GeoIP 数据库匹配国家/地区：

```ini
GEOIP, US, DIRECT
GEOIP, CN, DIRECT
```

### IP-ASN

匹配自治系统编号：

```ini
IP-ASN, 1234, DIRECT
```

### no-resolve 参数

当遇到 `GEOIP` 或 `IP-CIDR` 规则时，Surge 会发送 DNS 查询以检查请求的主机名是否为域名。使用 `no-resolve` 选项跳过 DNS 解析：

```ini
GEOIP, US, DIRECT, no-resolve
IP-CIDR, 172.16.0.0/12, DIRECT, no-resolve
```

## HTTP 规则 {#http-rule}

### USER-AGENT

匹配 User-Agent，支持通配符 `*` 和 `?`：

```ini
USER-AGENT, Instagram*, DIRECT
```

### URL-REGEX

匹配 URL 正则表达式：

```ini
URL-REGEX, ^http://google\.com, DIRECT
```

追加 `extended-matching` 参数同时测试 HTTP Host 请求头和 SNI：

```ini
URL-REGEX, ^https://example\.com, Proxy, extended-matching
```

## 进程规则（仅限 Mac） {#process-rule}

### PROCESS-NAME

匹配请求的进程名称，支持通配符 `*` 和 `?`：

```ini
PROCESS-NAME, Telegram, Proxy
PROCESS-NAME, /Applications/Safari.app/Contents/MacOS/Safari, DIRECT
```

## 逻辑规则 {#logical-rule}

### AND

所有子规则都匹配时触发：

```ini
AND, ((SRC-IP, 192.168.1.110), (DOMAIN, example.com)), DIRECT
```

### OR

任意子规则匹配时触发：

```ini
OR, ((SRC-IP, 192.168.1.110), (SRC-IP, 192.168.1.111)), DIRECT
```

### NOT

反转原始规则的评估结果：

```ini
NOT, ((SRC-IP, 192.168.1.110)), Proxy
```

逻辑规则支持嵌套：

```ini
AND, ((NOT, ((SRC-IP, 192.168.1.110))), (DOMAIN, example.com)), DIRECT
```

## 子网规则 {#subnet-rule}

### SUBNET

根据当前网络环境选择策略：

```ini
SUBNET, TYPE:WIRED, DIRECT
SUBNET, TYPE:CELLULAR, Proxy
SUBNET, SSID:MyHome, Proxy
```

子网表达式支持：

| 表达式 | 说明 |
|--------|------|
| `SSID:value` | 匹配 Wi-Fi SSID，支持通配符 |
| `BSSID:value` | 匹配 Wi-Fi BSSID，支持通配符 |
| `ROUTER:value` | 匹配路由器 IP 地址 |
| `TYPE:WIFI` | 匹配所有 Wi-Fi 网络 |
| `TYPE:WIRED` | 匹配所有有线网络 |
| `TYPE:CELLULAR` | 匹配所有蜂窝网络 |

## 杂项规则 {#misc-rule}

### 端口规则

```ini
DEST-PORT, 80-81, DIRECT        # 目标端口范围
IN-PORT, 6152, DIRECT            # 传入端口
SRC-PORT, >=50000, DIRECT        # 源端口（iOS 5.8.4+ / Mac 5.4.4+）
```

### SRC-IP

匹配客户端 IP 地址（仅适用于远程机器）：

```ini
SRC-IP, 192.168.20.0/24, DIRECT
```

### PROTOCOL

匹配请求协议：

```ini
PROTOCOL, HTTP, DIRECT
PROTOCOL, UDP, DIRECT
PROTOCOL, STUN, DIRECT
```

支持的值：`HTTP`、`HTTPS`、`TCP`、`UDP`、`DOH`、`DOH3`、`DOQ`、`QUIC`、`STUN`。

!!! warning "协议匹配说明"
    - `PROTOCOL,TCP` 涵盖 HTTP 和 HTTPS 连接
    - `PROTOCOL,UDP` 也可匹配 QUIC 流量
    - `DOH`/`DOH3`/`DOQ` 仅用于匹配 Surge 自身的加密 DNS 请求，需配合 `encrypted-dns-follow-outbound-mode=true` 使用
    - `STUN` 检测用于过滤 P2P 流量

### SCRIPT

使用 JavaScript 脚本作为规则：

```ini
SCRIPT, ScriptName, DIRECT
```

### CELLULAR-RADIO（仅限 iOS）

匹配当前蜂窝网络无线电技术：

```ini
CELLULAR-RADIO, LTE, DIRECT
```

可能的值：`GPRS`、`Edge`、`WCDMA`、`HSDPA`、`HSUPA`、`CDMA1x`、`CDMAEVDORev0`、`CDMAEVDORevA`、`CDMAEVDORevB`、`eHRPD`、`HRPD`、`LTE`、`NRNSA`、`NR`。

### HOSTNAME-TYPE

匹配主机名的形式（Mac 5.7.3+）：

```ini
HOSTNAME-TYPE, IPv4, DIRECT
HOSTNAME-TYPE, DOMAIN, Proxy
```

支持的关键字：`IPv4`、`IPv6`、`DOMAIN`、`SIMPLE`（不含点的主机名，如 `localhost`）。

### DEVICE-NAME / MAC-ADDRESS

匹配客户端设备名称或 MAC 地址（Mac 6.1.0+）：

```ini
DEVICE-NAME, My-Phone, DIRECT
MAC-ADDRESS, aa:bb:cc:dd:ee:ff, DIRECT
```

## 规则集 (Ruleset) {#ruleset}

### 内部规则集

Surge 提供两个内置规则集：

**SYSTEM** — 包含 macOS 和 iOS 系统请求的规则：

```ini
RULE-SET, SYSTEM, DIRECT
```

**LAN** — 包含局域网 IP 和 `.local` 后缀的规则：

```ini
RULE-SET, LAN, DIRECT
```

### 外部规则集

引用远程或本地文件中的规则列表。规则集文件每行包含一个不带策略的规则声明：

```ini
DOMAIN, exampleA.com
DOMAIN, exampleB.com
DOMAIN-SUFFIX, netflix.com
```

使用方式：

```ini
RULE-SET, https://example.com/social.list, Proxy, no-resolve, extended-matching
```

支持可选参数：
- `no-resolve`：跳过 DNS 解析
- `extended-matching`：域名规则同时匹配 SNI 和 HTTP Host

### 内联规则集（Mac 5.3.1+）

直接在配置文件中嵌入规则。内联规则集与独立文件共享相同的语法，并受益于相同的预处理/索引优化。

```ini
[Ruleset Streaming]
DOMAIN-SUFFIX,netflix.com
DOMAIN-SUFFIX,netflix.net
DOMAIN,netflixdnstest0.com

[Rule]
RULE-SET,Streaming,StreamingProxy
```

## 最终规则 (FINAL) {#final-rule}

`FINAL` 规则必须写在所有其他规则之后，作为默认策略：

```ini
[Rule]
DOMAIN-SUFFIX, company.com, ProxyA
DOMAIN-KEYWORD, google, DIRECT
GEOIP, US, DIRECT
FINAL, ProxyB
```

### 选项: dns-failed

当 DNS 查询失败时使用 `FINAL` 规则：

```ini
FINAL, Proxy, dns-failed
```
