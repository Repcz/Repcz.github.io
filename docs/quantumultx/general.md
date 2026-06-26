# 11. 配置文件`[general]`部分

配置文件主要有以下几个部分，点击左下角「编辑」按钮即可编辑；

进入编辑后，点击右上角箭头可快捷跳转：

|项目|备注|
|:-|:- |
|`[general]` |可以理解为通用设置|
|`[dns]`|[DNS相关设置](dns.md)|
|`[poilcy]`|[策略(策略组)](policy.md)|
|`[server_local]`|[本地节点](node.md?id=_22-添加本地节点)|
|`[server_remote]`|[远程节点订阅](node.md?id=_21-添加远程节点订阅)|
|`[filter_local]`|[本地分流规则](filter.md?id=_32-添加-本地规则)|
|`[filter_remote]`|[远程分流规则订阅](filter.md?id=_33-添加-远程引用规则)|
|`[rewrite_local]`|[本地重写规则](rewrite.md?id=_71-添加本地重写)|
|`[rewrite_remote]`|[远程重写订阅](rewrite.md?id=_72-添加远程重写)|
|`[task_local]`|[HTTP请求(本地脚本任务)](http-js.md)|
|`[http_backend]`|本地HTTP服务器|
|`[mitm]`|[主机名及证书相关](mitm.md)|

<!-- prettier-ignore -->
!!! 注意
    以下主要讲的是 `[general]` 区块下的内容，所以示例都以 `[general]` 开头表明在其之下，并不是让你每个参数字段前都加上 `[general]`。

### 11.1 配置文件图标
```
[general]
profile_img_url = https://avatars.githubusercontent.com/repcz
```

显示在「关联配置」的图标，如：

![profile_logo](Photo/profile_logo.webp){: width=300}

如上示例为 我的配置 的图标

### 11.2 资源解析器

```
[general]
resource_parser_url = https://raw.githubusercontent.com/KOP-XIAO/QuantumultX/master/Scripts/resource-parser.js
```

资源解析器的地址设置，资源解析器可用于自定义各类资源的转换

### 11.3 网络检查测试
```
[general]
network_check_url = http://wifi.vivo.com.cn/generate_204
server_check_url = http://cp.cloudflare.com/generate_204
; server_check_user_agent = Agent/1.0
server_check_timeout = 5000
```

- `network_check_url` 网络检查 URL 设置；
- `server_check_url` 代理服务器网络检查 URL 设置；
- `server_check_user_agent` 代理服务器检查的 `User Agent` 自定义设置；
- `server_check_timeout` 检测超时（毫秒），该值仅当 ≤ 5000 时生效。此值与整个检测过程（包括 DNS 查询、TCP 握手、TLS 握手及应用层 HTTP 会话）的总耗时进行比较，因此总耗时可能明显长于「TCP 握手」和「HTTP 会话时长」；

### 11.4 服务器 GEO 信息显示

节点页(首页)左上角的节点显示区域，可以修改配置文件(点左下角的编辑)，修改`geo_location_checker`参数，可以显示不同样式的信息

例如：

```
[general]
geo_location_checker=disabled
;geo_location_checker = http://ip-api.com/json/?lang=zh-CN, https://mirror.ghproxy.com/https://raw.githubusercontent.com/ConnersHua/RuleGo/master/Quantumult/Script/geo_location_checker.js
```

`disabled`表示关闭；

`#`或`;`或`//`表示注释，即起说明作用或者不生效，删除则表示启用，注意不同时启用两个`geo_location_checker`

![geochecker](Photo/geochecker.webp){: width=300}

### 11.5 「运行模式」与「临时禁用」

#### 11.5.1 `running_mode_trigger` 「运行模式」

```
[general]
; [蜂窝数据], [Wi-Fi], [SSID]
running_mode_trigger = filter, filter, LINK_22E171:all_proxy, LINK_22E172:all_direct
```

可选参数：

- `all_direct` 全部直连
- `all_proxy` 全部代理
- `filter` 规则分流

上述示例表示：在蜂窝数据使用分流 (第一个 `filter`)，在 Wi-Fi 使用分流 (第二个 `filter`)，在 SSID 为 `LINK_22E171` 时使用全部代理，在 SSID 为 `LINK_22E172` 时使用全部直连。

<!-- prettier-ignore -->
!!! 注意
    此时 `Rewrite` 与 `Task` 仍然生效。

#### 11.5.2 `ssid_suspended_list` 「临时禁用」
```
[general]
ssid_suspended_list = LINK_22E174, LINK_22E175
```

上述示例表示：在 SSID 为 `LINK_22E174` 与 `LINK_22E175` 时暂时禁用 Quantumult X

<!-- prettier-ignore -->
!!! 注意
    此时仅 `Task` 生效。

### 11.6 DNS 排除列表

```
[general]
dns_exclusion_list = *.cmpassport.com, *.jegotrip.com.cn, *.icitymobile.mobi, id6.me
```

不在该列表中的域名将会使用 Fake IP (CIDR 198.18.0.0/15)，并具有 `resolve-on-remote` 参数的效果，如想对指定域名使用真实 IP 地址可在此处设置。

另外，`dns_exclusion_list` 中的域名的查询可能不会遵循 [dns] 中的设置。

该功能类似于 Surge 的「Always Real IP」

### 11.7 排除路由
Quantumult X 将不会处理到 `excluded_routes` 的流量。

```
[general]
excluded_routes = 239.255.255.250/32
```

修改后最好重新启动设备。

### 11.8 域名拦截模式
```
[general]
dns_reject_domain_behavior = loopback
```

当域名在分流规则中被 `reject` 时，DNS 级别的拒绝行为由此参数控制。

可选参数有：

- `loopback`（默认）：返回回环 IP 响应（127.0.0.1 或 ::1）
- `no-error-no-answer`：返回 NOERROR 但不返回应答
- `nxdomain`：返回 NXDOMAIN（域名不存在）
- `none`：禁用 DNS 级别拒绝功能

!!! tip "切换说明"
    如果将已拒绝的域名改为非拒绝状态（通过修改配置、分流规则或策略），最多需要 10 秒生效（TTL 为 10）。

### 11.9 `UDP`相关设置

#### 11.9.1 `fallback_udp_policy`
当 UDP 请求经过规则模块以及策略模块后，所命中的服务器为 Quantumult X 所不支持 UDP Relay 的服务器或支持 UDP Relay 但未注明 `udp-relay=true` 的，则 `fallback_udp_policy` 会被使用。

```
[general]
fallback_udp_policy = reject
```

`fallback_udp_policy` 的值应为任意已开启 UDP Relay 的代理服务器 tag（名称）。默认值为 `reject`，任何不是服务器 tag（名称）的值将被忽略并使用 `reject`。

当 UDP 请求经过规则模块以及策略模块后所命中的节点为 Quantumult X 所不支持 UDP 转发的节点，或命中的节点虽支持 UDP 转发但节点配置未显示注明 `udp-relay=true` 的节点（例如：SS 或 SSR 且与服务器是否真实开启了 UDP 转发无关），则 `fallback_udp_policy` 会被使用。该参数默认值为 `reject`。

如一些海外游戏和语音使用 UDP 协议，而所使用的服务器不支持 UDP Relay 时，设置为 `reject` 将无法连接，如果需要调整该参数为 `direct` 或指定某个支持 UDP 的节点 tag，请务必清楚了解同一目标主机名 TCP 请求与 UDP 请求的源地址不同所造成的隐私及安全风险

#### 11.9.2 `udp_whitelist`

```
[general]
udp_whitelist = 53, 123, 1900, 80-443
```

用于设置 UDP 目标端口白名单，不在该列表中的 UDP 包将被丢弃并回发 ICMP（port unreachable）。空值表示允许所有端口。

一段范围用 `-` 字符连接。该设置与分流规则、策略及代理服务器端口无关。

#### 11.9.3 `udp_drop_list`
```
[general]
udp_drop_list = 1900, 80, STUN, QUIC
```

`udp_drop_list` 不会回发 ICMP (port unreachable) 消息。除端口号外，也支持协议名称如 `STUN`、`QUIC`。

仅被 `udp_whitelist` 允许的 UDP 包才能被 `udp_drop_list` 捕获。

#### 11.9.4 `icmp_auto_reply`

```
[general]
icmp_auto_reply = true
```

开启后 Quantumult X Tunnel 会自动回复 ICMP 请求（如 Ping）。

### 11.10 自定义 DoH 的 `User Agent`

```
[general]
doh_user_agent = Agent/1.0
```

自定义 DNS over HTTPS 的 `User Agent`

### 11.11 增强兼容性 SSID 列表

```
[general]
enhanced_compatibility_ssid_list = LINK_22E174, LINK_22E175
```

当全局选项「兼容性增强」（位于「其他设置」→「VPN」）关闭时，`enhanced_compatibility_ssid_list` 中的 SSID 将被考虑以开启兼容性增强。例如：兼容性增强全局关闭，但在 SSID 为 `LINK_22E174` 或 `LINK_22E175` 时自动开启。

