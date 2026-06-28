# 配置文件

> 本文档基于 [LOWERTOP/Shadowrocket](https://github.com/LOWERTOP/Shadowrocket/blob/main/README.md) 整理

Shadowrocket 配置文件（`*.conf`）是核心设置文件，包含通用参数、规则、Hosts、重写、HTTPS 解密、脚本等内容。[模块](advance.md#模块) 的写法与配置文件相同。

Shadowrocket 内置 `default.conf`，含国内外主要网站分流规则，跟随应用更新调整。误删可点击 **配置 > 恢复默认配置**。

## 配置管理

### 添加配置文件

**从 URL 下载**

`配置 > 右上角 ➕ > 粘贴配置链接 > 下载 > 点击配置文件 > 使用配置/编译配置`

Shadowrocket 兼容 Clash YAML 格式配置文件。含节点信息的标准 Clash 链接可同时导入配置和节点。

**从本地/云盘导入**

`配置 > 从云导入 > 选择配置文件`

### 配置文件操作

点击配置文件显示操作菜单：

| 操作 | 说明 |
|------|------|
| [使用配置](#使用配置) | 编译并启用，更新远程资源 |
| [编辑配置](#编辑配置) | UI 界面调整 |
| [编辑纯文本](#编辑纯文本) | 纯文本模式修改 |
| [编译配置](#编译配置) | 重新编译并更新远程资源 |
| [预览配置](#预览配置) | 查看纯文本（仅远程配置） |
| [更新配置](#更新配置) | 用远程版本覆盖本地 |
| [扩展配置](#扩展配置) | 建立配置包含关系 |
| [重新命名](#重新命名) | 重命名配置 |
| [导出配置](#导出配置) | 导出为文件 |

### 使用配置

编译并启用配置文件，同时更新该配置使用的远程[规则集](rule.md#规则集url)和[脚本](advance.md#脚本url)资源。

### 编辑配置

通过 UI 交互界面对配置文件进行调整。部分设置无 UI 界面，需使用[纯文本编辑](#编辑纯文本)。

### 编辑纯文本

纯文本模式修改配置。配置参考：

- [懒人配置](https://raw.githubusercontent.com/LOWERTOP/Shadowrocket/main/lazy.conf)
- [懒人配置（含策略组）](https://raw.githubusercontent.com/LOWERTOP/Shadowrocket/main/lazy_group.conf)

### 编译配置

重新编译配置文件，转译为符合当前版本的底层格式。更新规则集、脚本等远程资源。

### 预览配置

查看配置文件的纯文本格式。仅远程配置文件有此选项。

### 更新配置

用远程版本覆盖当前文件。同时更新规则集、脚本等资源。

<!-- prettier-ignore -->
!!! caution "更新配置会覆盖本地修改，本地修改过的远程配置请谨慎使用"

### 重新命名

重命名当前配置文件。

### 导出配置

将当前配置文件导出为文件。

### 扩展配置

配置间建立包含关系，满足同时使用多个配置的需求。

- 从配置 A 扩展出配置 B，B 包含 A（B 继承 A 的内容）
- 配置 B 优先级高于配置 A
- 修改/解除：`配置 ⓘ > 通用 > 包含配置`

---

## 通用参数

通用参数位于配置文件纯文本编辑模式的 `[General]` 字段，部分有 UI 界面，部分为[隐式参数](#隐式参数)。

- **UI 编辑**：`配置 ⓘ > 通用`
- **文本编辑**：点击配置文件 > 编辑纯文本 > `[General]`

语法格式：`key = value`

```ini
[General]
key = value
```

### 参数列表

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `bypass-system` | - | ~~旁路系统（旧版本参数，现已弃用）~~ |
| `skip-proxy` | - | 跳过代理接口，使用 TUN 接口接管。强制指定域名/IP 走 TUN 处理 |
| `tun-excluded-routes` | - | TUN 旁路路由，绕过指定 IP 范围 |
| `dns-server` | - | DNS 覆写，支持普通 DNS 和加密 DNS，参见 [DNS 设置](dns.md#dns-覆写) |
| `fallback-dns-server` | - | 备用 DNS，主 DNS 超时 2 秒后回退 |
| `ipv6` | `false` | 启用 IPv6 支持 |
| `prefer-ipv6` | `false` | 优先 IPv6 DNS 查询 |
| `private-ip-answer` | - | 域名解析返回私有 IP 时是否强制代理 |
| `tun-included-routes` | - | TUN 包含路由，添加较小路由表 |
| `always-real-ip` | - | TUN 处理 DNS 时返回真实 IP |
| `hijack-dns` | - | DNS 劫持，拦截硬编码 DNS 请求 |
| `include` | - | 包含配置（同[扩展配置](#扩展配置)） |

### 参数详解

#### `bypass-system`

旁路系统。旧版本参数，现已弃用。

#### `skip-proxy`

跳过代理接口，使用 TUN 接口接管。强制列表中的域名或 IP 交由 TUN 接口处理（非直连）。

启用 [TUN 模式](faq.md#代理类型) 时，所有连接强制走 TUN，此列表可忽略。

#### `tun-excluded-routes`

TUN 接口仅处理 TCP 协议。使用此选项绕过指定 IP 范围，让其他协议通过。

#### `dns-server`

DNS 覆写，覆盖系统 DNS。仅对直连域名生效，代理域名经代理服务器解析。

支持并发查询多个 DNS，采用最先返回的结果。详见 [DNS 设置](dns.md)。

#### `fallback-dns-server`

主 DNS 查询失败或超时 2 秒后回退。`system` 表示回退到系统 DNS。

#### `ipv6`

- `false`：不启用 IPv6
- `true`：启用 IPv6

即使不启用，若本地网络支持 IPv6 且节点域名支持 IPv6 解析，也会使用 IPv6 地址。可在 `[Host]` 中为节点域名指定 IP 解决。

#### `prefer-ipv6`

优先向 IPv6 DNS 查询 AAAA 记录。

#### `private-ip-answer`

不启用时，域名解析返回私有 IP 会视为劫持，强制走代理。

#### `tun-included-routes`

默认 Wi-Fi 接口路由较小，部分流量可能不经过 Shadowrocket。此选项添加较小路由表。

#### `always-real-ip`

TUN 处理 DNS 请求时返回真实 IP。

#### `hijack-dns`

劫持硬编码的 DNS 请求（如 Netflix 使用 Google DNS `8.8.8.8`）。

#### `include`

配置包含关系。当前配置包含另一配置，当前配置优先级更高。

---

## 隐式参数

<!-- prettier-ignore -->
!!! caution "隐式参数仅在纯文本 `[General]` 中设置，无 UI 界面，属于隐藏属性，建议谨慎设置"

语法格式：`key = value`

| 参数 | 说明 |
|------|------|
| `dns-direct-system` | 直连域名使用系统 DNS 查询 |
| `icmp-auto-reply` | Ping 数据包自动回复 |
| `always-reject-url-rewrite` | REJECT 策略在所有路由模式下生效 |
| `dns-direct-fallback-proxy` | 直连域名解析失败后使用代理 |
| `udp-policy-not-supported-behaviour` | UDP 匹配到不支持 UDP 的策略时的行为：`DIRECT` 或 `REJECT` |
| `stun-response-ip` | 返回虚假 IP 防止 WebRTC 泄露，如 `stun-response-ip=1.1.1.1` |
| `stun-response-ipv6` | IPv6 版 STUN 响应 IP |
| `compatibility-mode` | 网络兼容模式（见下文） |
| `always-ip-address` | 强制所有域名使用本地 DNS 解析 |
| `proxy-dns-server` | 指定 DNS 解析所有节点域名 |
| `close-if-proxy-chain-missing` | 代理链丢失时拒绝连接 |
| `ipv6-only-if-no-ipv4-dns` | 仅获取 IPv6 DNS 时视为 IPv6 Only 网络 |
| `block-quic` | QUIC 协议屏蔽策略 |
| `use-local-host-item-for-proxy` | 本地 Host 映射对代理生效 |
| `allow-dns-svcb` | 允许 DNS SVCB 查询 |
| `allow-dns-all` | 允许所有 DNS 查询类型正常转发 |

### `compatibility-mode`

| 值 | 效果 |
|----|------|
| `0` | 自动/禁用 |
| `1` | Proxy with Loopback Address |
| `2` | Proxy Only |
| `3` | TUN Only（等同于启用 [TUN 模式](faq.md#代理类型)） |
| `4` | Proxy without Loopback Address |
| `5` | No Default Route（等同于启用 [兼容模式](faq.md#兼容模式)） |

优先级高于软件设置内的相关选项。

### `block-quic`

| 值 | 说明 |
|----|------|
| `all-proxy` | 仅阻断走代理连接的 QUIC |
| `all` | 阻断所有连接（包括直连）的 QUIC |
| `always-allow` | 始终允许 QUIC |

### 其他隐式参数说明

- **`dns-direct-system`**：直连域名使用系统 DNS 查询。`true` 启用
- **`icmp-auto-reply`**：Ping 自动回复。`true` 启用
- **`always-reject-url-rewrite`**：使 URL 重写的 REJECT 策略在所有路由模式下生效。`true` 启用
- **`dns-direct-fallback-proxy`**：直连域名解析失败后走代理。`true` 启用
- **`udp-policy-not-supported-behaviour`**：UDP 流量匹配到不支持 UDP 的节点策略时，可选 `DIRECT`（直连）或 `REJECT`（拒绝）
- **`stun-response-ip`** / **`stun-response-ipv6`**：返回虚假 IP 防止 WebRTC 真实 IP 泄露，如 `stun-response-ip=1.1.1.1`。设置后将忽略设置中的[禁用 STUN](faq.md#禁用-stun) 开关
- **`always-ip-address`**：强制所有域名使用本地 DNS 解析
- **`proxy-dns-server`**：使用指定 DNS 解析所有节点域名。未设置时节点域名默认使用 `dns-server` 解析
- **`close-if-proxy-chain-missing`**：代理链中的中转节点丢失时：`true` 拒绝连接，`false` 跳过中转直连落地节点
- **`ipv6-only-if-no-ipv4-dns`**：当设备仅获取到 IPv6 DNS 时视为 IPv6 Only 网络
- **`use-local-host-item-for-proxy`**：使本地 Host 映射对代理生效。默认代理类 DNS 在远端执行
- **`allow-dns-svcb`**：允许 DNS SVCB 查询。默认禁止以强制系统执行 A 记录查询
- **`allow-dns-all`**：允许所有 DNS 查询类型正常转发。默认开启，关闭后非 A/AAAA 查询返回空值

---

## 编写本地节点

<!-- prettier-ignore -->
!!! caution "编写本地节点仅用于兼容其他类型配置文件，不应作为添加节点的优先选择"

在配置文件纯文本中编写本地节点：

### Shadowsocks

```ini
节点名称 = ss,地址,端口,password=密码,method=aes-256-cfb,obfs=websocket
```

### Vmess

```ini
节点名称 = vmess,地址,端口,password=密码,alterId=0,method=auto,obfs=websocket,tfo=1
```

### VLESS

```ini
节点名称 = vless,地址,端口,password=密码,tls=true,obfs=websocket,peer=example.com
```

### HTTP/HTTPS/Socks5

```ini
节点名称 = http,地址,端口,用户,密码
节点名称 = https,地址,端口,用户,密码
节点名称 = socks5,地址,端口,用户,密码
节点名称 = socks5-tls,地址,端口,用户,密码,skip-common-name-verify=true
```

### Trojan

```ini
节点名称 = trojan,地址,端口,password=密码,allowInsecure=1,peer=example.com
```

### Hysteria

```ini
节点名称 = hysteria,地址,端口,auth=密码,obfsParam=混淆,protocol=协议,udp=1,peer=example.com,alpn=h2,upmbps=100,downmbps=100
```

### Hysteria2

```ini
节点名称 = hysteria2,地址,端口,auth=密码,obfsParam=混淆,udp=1,peer=example.com,alpn=h3
```

### TUIC

```ini
节点名称 = tuic,地址,端口,password=密码,udp=1,user=uuid值,peer=example.com,alpn=h2
```

### Juicity

```ini
节点名称 = juicity,地址,端口,password=密码,udp=1,user=uuid值,peer=example.com,alpn=h2
```

### WireGuard

```ini
节点名称 = wireguard,地址,端口,privateKey=私钥,publicKey=公钥,ip=子网IP,udp=1,dns=1.1.1.1,mtu=1350,keepalive=40,reserved=1/2/3
```

### Snell v2

```ini
节点名称 = snell,地址,端口,password=密码,udp=1,obfs=http,obfs-host=example.com,obfs-uri=/abc
```

---

> 继续阅读：[**规则系统** &rarr;](rule.md)
