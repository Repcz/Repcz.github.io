# 节点与订阅

> 本文档基于 [LOWERTOP/Shadowrocket](https://github.com/LOWERTOP/Shadowrocket/blob/main/README.md) 整理

## 添加节点

Shadowrocket 兼容多种服务器订阅格式及代理协议类型。

<!-- prettier-ignore -->
!!! tip "添加或更新订阅出现异常时，请参见[订阅异常](faq.md#订阅异常)"

### 扫码添加

`首页 > 左上角 > 扫码添加`

需开启摄像头权限。

### 剪贴板导入

复制节点链接（如 `trojan://*`、`vmess://*`、`vless://*` 等），打开 Shadowrocket 时自动识别导入。

需开启剪贴板读取权限。

### 订阅添加

`首页 > 右上角 ➕ > 类型 Subscribe > URL 栏输入订阅链接 > 保存`

- 订阅链接后加 `#1`、`#2`、`#3` 可重复添加同一订阅
- 长按订阅可选**隐藏**功能，等同于在订阅编辑中设置**最大值**为 `0`

### 手动添加

`首页 > 右上角 ➕`，选择对应协议类型，填写配置信息并保存。

### WireGuard 节点

`首页 > 右上角 ➕ > 类型 WireGuard`，填写配置信息。

或直接复制 WireGuard 配置格式，打开 Shadowrocket 时自动弹出添加对话框：

```ini
[Interface]
PrivateKey = xxxxxx
Address = 172.16.0.2/32
DNS = 1.1.1.1
MTU = 1420
[Peer]
PublicKey = xxxxxx
AllowedIPs = 0.0.0.0/0
AllowedIPs = ::/0
Endpoint = engage.cloudflareclient.com:2408
Reserved = 12,34,56
```

## 协议类型

Shadowrocket 支持的协议类型（可在 `首页 > 右上角 ➕ > 类型` 中查看）：

| 类型 | 说明 |
|------|------|
| Subscribe | 订阅 |
| Shadowsocks | SS |
| ShadowsocksR | SSR |
| Vmess | VMess |
| VLESS | VLESS |
| Relay | 中继 |
| Socks5 | SOCKS5 |
| Socks5 Over TLS | SOCKS5 over TLS |
| HTTP | HTTP |
| HTTPS | HTTPS |
| HTTP2 | HTTP/2 |
| Trojan | Trojan |
| Hysteria | Hysteria |
| Hysteria2 | Hysteria2 |
| AnyTLS | AnyTLS |
| TUIC | TUIC |
| Juicity | Juicity |
| SSH | SSH 隧道 |
| WireGuard | WireGuard |
| Snell | Snell |
| Mieru | Mieru |
| OpenConnect | OpenConnect |
| TrustTunnel | TrustTunnel |
| Brook | Brook |
| Lua | Lua 脚本 |

各协议的纯文本格式参见[编写本地节点](config.md#编写本地节点)。

## 更新订阅节点

- **右滑订阅** > 更新
- **首页更新按钮** 🔄
- **设置 > 订阅 > 打开时更新**（刷掉后台重新打开自动更新）
- **设置 > 订阅 > 自动后台更新**（需开启系统后台 App 刷新，周期 1-24 小时）
- **快捷指令**：使用「更新订阅」快捷指令实现自动化
- **长按应用图标** > 更新订阅
- **自动更新**：[详见自动更新](faq.md#自动更新)

## 节点排序

`设置 > 订阅 > 根据 Ping 排序`

根据节点延迟测试结果由低到高自动排序。

## 节点分享与整理

### 分享节点

- **长按节点** > 拷贝（分享节点链接）
- **左滑节点** > 二维码（生成节点二维码）
- **节点 ⓘ > 滑动至底部** > 多种分享方式
- **编辑模式**：点击 `•••` 勾选节点 > 左上角复制（批量分享）

<!-- prettier-ignore -->
!!! warning "节点二维码及 URL 缺乏统一标准，导出至其他代理工具时可能丢失部分信息"

### 整理节点

| 操作 | 方法 |
|------|------|
| **调整顺序** | 点击 `•••` 或长按首页右上角 `➕`，按住 `≡` 拖动 |
| **节点分类** | 编辑模式勾选节点 > 左上角**折叠** > 命名新分类 |
| **删除节点** | 编辑模式 > 左上角**删除**（可删除全部或超时节点） |
| **滑动菜单** | 右滑订阅：测试/更新；右滑节点：测试/复制 |

## 订阅节点筛选

`首页 > 订阅 ⓘ > 过滤`

### 节点筛选（正则表达式）

| 目的 | 表达式 |
|------|--------|
| 保留含关键词 A **和** B 的节点 | `/(?=.*(A))^(?=.*(B))^.*$/` |
| 保留含关键词 A **或** B 的节点 | `A\|B` |
| 排除含关键词 A **或** B 的节点 | `/^((?!(A\|B)).)*$/` |
| 保留 A 并排除 B | `/(?=.*(A))^((?!(B)).)*$/` |

> 分组与代理分组中的正则写法相同，但需删除前后 `/`

### 批量整理（脚本）

在订阅筛选输入框中使用脚本批量修改节点：

| 目的 | 脚本 |
|------|------|
| 关键词 a 替换为 b | `$server.title=$server.title.replace(/关键词a/g,'关键词b')` |
| 名称前加前缀 | `$server.title='abc'+$server.title` |
| 名称后加后缀 | `$server.title=$server.title+'abc'` |
| 批量开启 RESERVED | `$server.reserved="1,40-60,30-50"` |
| 批量开启 MUX | `$server.mux=1` |
| 批量设置代理链 | `$server.chain="订阅名称/节点名称"` |
| 批量设置代理链(UUID) | `$server['dialer-proxy']="UUID值"` |
| 按地址加地区标识 | `if (/关键词/i.test($server.host)) { $server.title = '🇭🇰'+ $server.title }` |

## 代理通过/代理链

当前代理通过另一个代理进行连接，支持多级链式代理。

- 节点 A 的 `ⓘ > 代理通过` 选择节点 B，流量走向：`Client > B > A > Web server`
- 支持使用分组或代理分组作为中转
- 支持[批量修改](node.md#批量整理脚本)订阅节点的代理链
- 取消：节点 `ⓘ > 代理通过 > 右上角取消 > 保存`

---

> 继续阅读：[**配置文件** &rarr;](config.md)
