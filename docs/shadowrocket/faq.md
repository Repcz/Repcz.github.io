# 常见问题与附录

> 本文档基于 [LOWERTOP/Shadowrocket](https://github.com/LOWERTOP/Shadowrocket/blob/main/README.md) 整理

---

## 设置页面

### 延迟测试方法

详见[连通性测试](index.md#连通性测试)。

### 小组件

`设置 > 小组件`

- **服务器节点**：添加 6 个常用节点，展开查看
- **显示 Ping 值**：长按小组件中心测试延迟
- **根据 Ping 排序**：测试后自动排序

**添加方法**：

- **Today 小组件**：负一屏 > 编辑 > 添加 Shadowrocket（< iOS 18）
- **屏幕小组件**：长按屏幕 > `➕` > Shadowrocket（≥ iOS 17）
- 更新后找不到可尝试重启手机

### 外观

`设置 > 外观`

支持自定义配色，可分享和安装配色方案。配色安装：复制配色链接，在配置页面点击 `➕` 粘贴安装。

配色项目示例：

```ruby
NavigationBarColor / 导航栏背景色
NavigationBarTextColor / 导航栏文字颜色
TabBarColor / 标签栏背景色
TableBackgroundColor / 表格背景颜色
PingSuccessTextColor / 测试Ping成功文字颜色
PingTimeoutTextColor / 测试Ping超时文字颜色
```

### 按需求连接

| 选项 | 说明 |
|------|------|
| **始终开启** | VPN 保持连接，断开/重启自动重连。脚本/模块需求多时建议开启 |
| **按需求连接** | 根据规则自动切换 VPN 状态 |
| **睡眠时断开** | 设备进入睡眠时断开 VPN |
| **显示断开信息** | VPN 断开时显示通知 |

> 同时开启始终开启和按需求连接时，仅始终开启生效

### 隧道

| 选项 | 说明 |
|------|------|
| **强制路由** | 隧道路由优先于本地路由 |
| **包括所有网络** | 路由所有网络流量 |
| **包括本地网络** | 包含 AirPlay、AirDrop 等本地流量 |
| **包括 APNs** | 包含苹果推送通知服务 |
| **包括蜂窝服务** | 包含 VoLTE、Wi-Fi 通话等 |

### 兼容模式

`设置 > 代理 > 兼容模式`

解决 HomeKit 摄像头 RTSP 回包问题。开启效果等同于 `compatibility-mode=5`。

### 前置代理

`设置 > 代理 > 前置代理`

所有流量先通过 HTTP/SOCKS5 代理转发，再根据规则向节点发送请求。

### 代理共享

将代理设置分享给局域网/热点设备使用。

- **局域网**：A 设备启用共享，B 设备设置 HTTP 代理为 A 的 IP 和端口
- **热点**：先连接热点形成局域网，再按局域网方式设置

<!-- prettier-ignore -->
!!! tip "开启代理共享时建议保持屏幕常亮或连接充电器"

### 代理类型

| 类型 | 说明 |
|------|------|
| **HTTP** | 系统代理模式，不支持的程序由 TUN 接管 |
| **None** | TUN 模式，全部请求通过 TUN 接口处理（等同于 `compatibility-mode=3`） |

### 开启 UDP 转发

- `设置 > UDP > 启用转发`
- `订阅 ⓘ > UDP转发`
- `节点 ⓘ > UDP转发`

### 禁用 STUN

`设置 > UDP > 禁用STUN`

阻止 WebRTC 通过 STUN 获取公网 IP，防止 IP 泄露。

> 若配置中使用 `stun-response-ip` 命令，此处开关将被忽略

### Tailscale

≥ 2.2.89 版本支持 Tailscale 全局隧道模组。

- **启用**：处理 tailnet 流量
- **认证密钥**：填写 Tailscale 管理控制台的认证密钥
- **控制服务器 URL**：自定义控制服务器地址
- **出口节点**：通过 Tailscale 网络中设备转发流量
- **始终使用 DERP**：强制通过 DERP 中继转发

可配合规则策略使用：

```ini
DOMAIN-WILDCARD,tail*.ts.net,TAILSCALE
DOMAIN-SUFFIX,ts.net,TAILSCALE
```

> `tun-excluded-routes` 中包含 `100.64.0.0/10` 可能影响 Tailscale

### 权限

| 权限 | 用途 |
|------|------|
| **位置** | 读取 Wi-Fi SSID（场景功能需要） |
| **通知** | 脚本结果、订阅更新、VPN 断开等通知 |
| **剪贴板** | 自动识别剪贴板中的代理 URL |

### 隐藏 VPN 图标

`设置 > 排除路由 0.0.0.0/31`

利用系统漏洞实现，不推荐开启（可能导致网络异常）。

### GeoIP 数据库

`设置 > GeoLite2 数据库`

软件已内置常规 GeoIP 数据库，一般无需设置。

**自定义数据库**：

| 来源 | 地址 |
|------|------|
| Loyalsoldier | `https://raw.githubusercontent.com/Loyalsoldier/geoip/release/Country.mmdb` |
| Hackl0us | `https://github.com/Hackl0us/GeoIP2-CN/raw/release/Country.mmdb` |
| Masaiki | `https://github.com/Masaiki/GeoIP2-CN/raw/release/Country.mmdb` |
| P3TERX (ASN) | `https://github.com/P3TERX/GeoLite.mmdb/raw/download/GeoLite2-ASN.mmdb` |

### 自动更新

包含配置、模块、订阅、GeoLite2 数据库的自动更新。需在系统设置中为 Shadowrocket 启用后台 App 刷新。

**配置自动更新**：

- **自动后台更新**：1-7 天间隔
- **更新提醒**：需开启推送通知
- 本地配置文件仅更新规则集/脚本等资源

**订阅自动更新**：

- **自动后台更新**：1-24 小时间隔
- **DNS**：指定 HTTPS DNS 解析订阅链接域名
- **根据 PING 排序**：按延迟排序
- **保持代理通过**：保持代理链设置
- **发送 HWID**：发送 `X-HWID` 标头

### 温和策略机制

启用后切换策略不打断已有 TCP 连接，仅新连接使用新策略。不启用则切换会中断旧连接。

---

## 数据页面

### 代理日志

`数据 > 代理 > 启用日志记录`

记录网络活动详情：请求 URL、规则策略、传输协议、发送时间。

- 显示 `MITM` 表示已启用解密
- 点击记录查看详情，`•••` 可添加规则
- 自动删除：自动清理 7 天前日志

### DNS 日志

`数据 > DNS > 启用日志记录`

记录 DNS 查询详情：域名、结果、响应时间、处理 DNS。

- 旗帜根据返回 IP 地理信息显示
- 响应时间超过 2 秒说明触发回退机制

### iCloud 自动同步

`数据 > iCloud > 自动同步`

同步节点、配置、模块、脚本等数据。

<!-- prettier-ignore -->
!!! tip "多设备建议：主设备开启同步，副设备关闭同步，仅在需要时手动操作"

**节点恢复问题**：如果删除节点后自动恢复，尝试 `数据 > iCloud > 删除 iCloud 备份` 和 `同步服务器节点`。

### 节点数据管理

| 操作 | 说明 |
|------|------|
| **导出节点** | 导出为 JSON 文件 |
| **导入节点** | 从 JSON 文件导入 |
| **删除本地节点** | 一键删除所有节点 |

### 流量统计信息

`数据 > 统计`

记录连接时间、Wi-Fi/蜂窝上下行流量、分流柱形图。

- **启用存档**：单独记录每次连接流量
- 订阅支持显示流量信息：订阅链接添加 `subscription-userinfo` 响应头或在 Base64 编码前添加 `STATUS=xxx`

---

## 常见问题

### 自动切换节点

**方法一**：`首页 > 全局路由 > 分组 > 简单模式`

**方法二**：`配置 ⓘ > 代理分组 > 添加 url-test 类型分组`

> 节点不稳定时可同时[开启回退](index.md#启用回退)

### 订阅异常

**错误原因**：

| 提示 | 原因 |
|------|------|
| `forbidden` | 订阅被重置或 token 错误 |
| `not found` | 订阅路径或链接错误 |
| `service unavailable` | 域名错误或被运营商屏蔽 |
| `SSL 错误` | 无法建立安全连接 |

**解决方法**：

- 全局路由选代理，使用其他节点更新
- 切换网络后重试
- 检查订阅链接是否错误或失效

### 节点旗帜

节点优先根据备注名称匹配旗帜，其次由地址解析 IP 判断国家/地区。

- 节点 `ⓘ > 地址栏图标`：手动修改
- 旗帜 Emoji 放在节点备注开头，保存时自动显示对应旗帜且不显示 Emoji

### 节点感叹号

节点显示 `⚠` 的原因：节点使用 TLS，地址是 IP，却没有设 SNI。

Shadowrocket 会主动开启**允许不安全**，跳过 TLS 证书验证。建议使用自签名证书并导入信任，或及时续订服务器证书。

### 微信转圈

微信一直显示**连接中/收取中**：

- 微信分流走直连
- `配置 ⓘ > 通用 > 启用IPv6 > 关闭`

本质是部分地区运营商 IPv6 不稳定或路由器设置不当导致。

### 模块消失

模块页面已开启**保存到 iCloud** 时：

1. 检查 iCloud 中 Shadowrocket 同步是否开启
2. 检查 `文件 app > iCloud 云盘 > Shadowrocket` 是否包含 `Modules` 和 `Script` 文件夹
3. iCloud 缓存被清理时需等待自动或手动下载

### 模块失效

- 切换配置文件需重新开启解密，使用[证书模块](advance.md#证书模块)可避免
- 尝试[使用配置](config.md#使用配置)/[编译配置](config.md#编译配置)重新拉取远程资源

### VPN 自动断开

iOS 15 以下系统、复杂请求、加解密、运行脚本等可能导致 NE 内存占用过高而断开。

**解决方法**：`设置 > 按需求连接 > 始终开启 > 启用`

### 检测代理

某些 App 提示需关闭代理：

`设置 > 代理类型 > 选择 None`（TUN 模式接管代理）

但对直接检测 VPN 状态或有其他检测手段的 App 无效。

### 编译原因

- Shadowrocket 2.2.29 之前使用 Xcode 13.2.1 编译
- 2023年4月后需 Xcode 14 编译，最低要求 iOS 12
- 2.2.30 起设置最低 iOS 12，低版本用户可安装 2.2.28

---

## URL Schemes

Shadowrocket 支持 `X-callback-urls`。支持 `autoclose=true` 参数，动作触发后退出应用。

| 操作 | URL |
|------|-----|
| 启动 VPN | `shadowrocket://connect` 或 `shadowrocket://open` |
| 停止 VPN | `shadowrocket://disconnect` 或 `shadowrocket://close` |
| 切换开关 | `shadowrocket://toggle` |
| 连通性测试 | `shadowrocket://connectivity-test` |
| 使用特定节点 | `shadowrocket://select?s={节点名称}` |
| 添加订阅/节点 | `shadowrocket://add/{url}` |
| 更新订阅 | `shadowrocket://update-subs` |
| 安装/使用配置 | `shadowrocket://config/add/{url}` |
| 安装模块 | `shadowrocket://install?module={url}` |
| 切换代理模式 | `shadowrocket://route/proxy` |
| 切换配置模式 | `shadowrocket://route/config` |
| 切换直连模式 | `shadowrocket://route/direct` |
| 切换场景模式 | `shadowrocket://route/scene` |
| 安装配色 | `shadowrocket://color?{配色设置}` |
