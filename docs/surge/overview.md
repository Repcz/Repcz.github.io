# 概述与核心概念

## 核心工作流 {#workflow}

Surge 的核心工作流由四项主要能力组成：

- **接管**：允许用户接管设备发送的网络连接。支持代理服务和虚拟网卡（VIF）接管。
- **处理**：允许用户修改被接管的网络请求和响应，包括 URL 重定向、本地文件映射、JavaScript 自定义修改等。
- **转发**：网络请求被接管后，可将其转发到其他代理服务器。转发可以是全局的，也可使用灵活的规则系统确定出站策略。
- **截获**：截获并保存网络请求和响应中的特定数据，通过 MITM 解密 HTTPS 流量。

## 特性 {#features}

- **高性能、稳定、高效**：以工业级稳定性、最少系统资源处理所有网络流量
- **灵活的规则系统**：基于域名、IP-CIDR、GeoIP 等编写转发规则
- **HTTPS 解密**：通过 MITM 解密 HTTPS 流量，内置证书生成器
- **本地 DNS 映射**：支持自定义 DNS 映射，包括通配符、别名和自定义 DNS 服务器
- **策略组**：多个代理归为一组，支持自动测速、SSID 选择、手动选择等
- **HTTP 重写**：将 HTTP/HTTPS 请求重写到另一个 URL，或阻止请求
- **远程面板**：Dashboard 可通过 USB 或网络连接远程 Surge 实例
- **完整的 IPv6 支持**

### Surge Mac 独占特性 {#mac-features}

- **增强模式**：通过虚拟网络接口处理所有网络流量
- **计费网络模式**：控制允许哪些应用访问互联网
- **网关模式**：配置为三层网关，处理同一网络中其他设备的流量

### Surge iOS 独占特性 {#ios-features}

- 所有功能均可在蜂窝网络上使用
- 捕获所有 HTTP/HTTPS/TCP 流量，即使应用不遵循系统代理设置
- 覆盖系统 DNS 设置，同时查询所有 DNS 服务器提升性能
- 通过 Wi-Fi 或 USB 连接 Surge Dashboard

## 组件 {#components}

### 代理服务器 (Proxy Server) {#proxy-server}

Surge 内置一个多功能的代理服务器，支持 HTTP/HTTPS/SOCKS5/SOCKS5-TLS 协议，可监听多个端口为局域网设备提供代理服务。

### 虚拟网卡 (Virtual Interface, VIF) {#vif}

Surge 创建一个虚拟网卡来接管设备的所有网络流量。该虚拟网卡由 Surge 完全控制，支持：

- **增强模式**（仅限 Mac）：创建虚拟网卡处理所有 TCP/UDP 流量
- **TUN 模式**（仅限 iOS）：创建虚拟网卡处理流量

### DNS 解析器 {#dns-resolver}

Surge 拥有自己的 DNS 客户端实现：

- 同时向所有上游 DNS 服务器发起查询，使用最先返回的响应
- 支持 DNS over HTTPS (DoH)、DNS over HTTP/3 (DoH3)、DNS over QUIC (DoQ)
- 通过虚假 IP (198.18.0.0/15) 实现 DNS 劫持

## 配置文件结构 {#profile-structure}

Surge 使用 INI 风格的配置文件，包含多个段落：

```ini
[General]
# 通用设置
loglevel = notify
skip-proxy = 192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12
dns-server = system, 8.8.8.8, 1.1.1.1

[Proxy]
# 代理服务器声明
ProxyA = https, example.com, 443, username, password
ProxyB = ss, example.com, 8000, encrypt-method=chacha20-ietf-poly1305, password=1234

[Proxy Group]
# 策略组定义
Auto = url-test, ProxyA, ProxyB, url=http://www.gstatic.com/generate_204, interval=600

[Rule]
# 规则定义
DOMAIN-SUFFIX, google.com, Auto
GEOIP, CN, DIRECT
FINAL, ProxyA

[MITM]
# HTTPS 解密设置
hostname = *
enable = true

[Host]
# DNS 映射
*.dev = 127.0.0.1

[URL Rewrite]
# URL 重写
^http://example\.com https://example.com header

[Header Rewrite]
# 请求头重写
http-request ^http://example.com header-add DNT 1

[Script]
# 脚本
cron = type=cron, cronexp="0 2 * * *", script-path=cron.js

[Panel]
# 信息面板（仅限 iOS）
Info = title="面板", content="内容", style=info

[Module]
# 模块声明
# 模块通过 UI 安装和管理

[Map Local]
# 本地映射
^http://example\.com/api data-type=text data="{}" status-code=200
```

### 段落说明 {#sections}

| 段落 | 描述 |
|------|------|
| `[General]` | 通用配置选项 |
| `[Proxy]` | 代理服务器定义 |
| `[Proxy Group]` | 策略组定义 |
| `[Rule]` | 流量路由规则 |
| `[MITM]` | HTTPS 解密配置 |
| `[Host]` | 本地 DNS 映射 |
| `[URL Rewrite]` | URL 重写规则 |
| `[Header Rewrite]` | 请求/响应头重写 |
| `[Body Rewrite]` | 请求/响应体重写 |
| `[Script]` | JavaScript 脚本 |
| `[Panel]` | 信息面板（iOS） |
| `[Map Local]` | 本地模拟响应 |
| `[Port Forwarding]` | 端口转发 |
| `[Managed]` | 托管配置锁定 |
| `[Keystore]` | 密钥和证书存储 |

## 配置管理 {#config-management}

### 托管配置 (Managed Profile) {#managed-profile}

托管配置允许配置文件由远程服务器管理，用户界面中的某些设置可以被锁定，防止最终用户修改。相关设置可在 `[Managed]` 段落中配置。

### 模块 (Module) {#module-overview}

模块是一组配置片段的集合，包含 `[General]`、`[Rule]`、`[Script]`、`[MITM]`、`[Host]`、`[URL Rewrite]`、`[Header Rewrite]`、`[Body Rewrite]`、`[Panel]`、`[Map Local]` 等段落。模块可通过 UI 安装和管理，用于扩展 Surge 的功能。
