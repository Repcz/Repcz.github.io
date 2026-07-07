# 高级功能

## 增强模式（仅限 Mac） {#enhanced-mode}

增强模式通过创建一个虚拟网卡 (VIF) 接管设备的所有 TCP 和 UDP 流量，使不遵循系统代理设置的应用程序也能被 Surge 处理。

```ini
[General]
enhanced-mode = true
```

### 相关选项

#### tun-excluded-routes

绕过特定 IP 范围，允许流量不经过 Surge VIF：

```ini
[General]
tun-excluded-routes = 192.168.0.0/16, 10.0.0.0/8
```

#### tun-included-routes

添加更小的路由确保流量通过 Surge VIF：

```ini
[General]
tun-included-routes = 0.0.0.0/1, 128.0.0.0/1
```

支持 IPv6 CIDR 块。

## 端口转发（iOS 5.14.3+ / Mac 5.10.0+） {#port-forwarding}

监听特定本地端口并将 TCP 请求转发到特定主机。即使未启用系统代理或增强模式也可独立使用：

```ini
[Port Forwarding]
0.0.0.0:6841 localhost:3306 policy=SQL-Server-Proxy
```

`policy` 参数可选，未指定则使用标准代理匹配。

## 信息面板（仅限 iOS 4.9.3+） {#panel}

自定义信息面板显示相关信息。

### 静态模式

```ini
[Panel]
PanelA = title="面板标题", content="面板内容\n第二行", style=info
```

支持 `style`：`good`、`info`、`alert`、`error`。

### 动态模式

通过脚本更新面板内容：

```ini
[Panel]
PanelB = title="面板标题", content="面板内容\n第二行", style=info, script-name=panel, update-interval=60

[Script]
panel = script-path=panel.js, type=generic
```

```javascript
$httpClient.get("https://api.my-ip.io/ip", function(error, response, data) {
    $done({
        title: "External IP Address",
        content: data,
    });
});
```

### 更多自定义

```ini
[Panel]
PanelC = title="自定义", content="内容", icon="bolt.horizontal.circle.fill", icon-color="#FF9500"
```

- 不传入 `style` 时不显示图标，可传入 `icon`（SF Symbol Name）
- 传入 `icon-color`（HEX 颜色码）控制图标颜色
- `update-interval` 使面板自动更新（秒）

## 模块 (Module) {#module}

模块是一组配置片段的集合，用于扩展 Surge 功能。可通过 UI 安装和管理。

### 安装模块

1. **首页** → **通用** → **模块** → 点击 **模块** 按钮
2. 下滑至 **安装的模块** 最下方，点击 **安装新模块**
3. 填入模块 raw 链接并保存
4. 等待外部资源下载
5. 点击 **调整生效顺序** 调整模块顺序

!!! tip "注意"
    新安装模块不会自动启用，需手动勾选。

### 模块包含的段落

模块可覆盖以下段落：

- `[General]`、`[MITM]`：支持覆盖（`key = value`）、追加（`key = %APPEND% value`）、插入（`key = %INSERT% value`）
- `[Rule]`、`[Script]`、`[URL Rewrite]`、`[Header Rewrite]`、`[Host]`：新行插入到原始内容顶部
- `[Ruleset *]`：内联规则集补丁
- `[WireGuard *]`：WireGuard 策略追加

模块中的规则只能使用内部策略：`DIRECT`、`REJECT`、`REJECT-TINYGIF`。

### 模块元数据

```ini
#!name=模块名称
#!desc=模块描述
#!system=mac        # 可选，限制生效平台
#!arguments=hostname=example.com&enable_mitm=true  # 可选，自定义参数
#!requirement=CORE_VERSION>=20  # 可选，版本要求
```

## 网关模式（仅限 Mac） {#gateway-mode}

Surge Mac 可配置为三层网关，处理同一网络中其他设备的网络流量：

```ini
[General]
gateway-mode = true
```

## \[General\] 杂项选项 {#general-options}

### 日志与调试

```ini
loglevel = notify          # verbose / info / notify / warning
show-error-page = true     # Mac 5.8.0+，显示错误页面
show-error-page-for-reject = true  # 为 REJECT 显示错误页面
```

### 网络

```ini
ipv6 = true                # 启用完整 IPv6 支持
ipv6-vif = auto            # off / auto / always
dns-server = system, 8.8.8.8
skip-proxy = 192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12
exclude-simple-hostnames = true  # 简单主机名绕过代理
```

### DNS

```ini
always-real-ip = *.local   # 返回真实 IP
hijack-dns = *:53          # 劫持 DNS 查询
use-local-host-item-for-proxy = true  # 代理请求使用本地 DNS
```

### HTTP

```ini
full-header-mode = true    # 暴露完整请求头数组
```

### 远程访问

```ini
external-controller-access = key@0.0.0.0:6165
http-api = key@0.0.0.0:6166
http-api-tls = true        # 使用 HTTPS
http-api-web-dashboard = true  # 启用 Web Dashboard
```

### 测试

```ini
internet-test-url = http://www.gstatic.com/generate_204
proxy-test-url = http://www.gstatic.com/generate_204
test-timeout = 5
```

## Host List 参数类型 {#host-list}

Surge 中多处使用 Host List 类型参数（如 `skip-proxy`、`always-real-ip`、`hostname` 等）。

### 语法规则

| 模式 | 示例 | 说明 |
|------|------|------|
| 域名 | `example.com` | 匹配单个域名 |
| 通配符 | `*google.com` | 匹配所有包含 google.com 的域名 |
| IP 地址 | `192.168.2.1` | 匹配特定 IP |
| IP 范围 | `192.168.2.*` | 通配符 IP 范围 |
| CIDR | `192.168.2.0/24` | CIDR 表示法 |
| 排除 | `-*.apple.com` | 以 `-` 为前缀排除 |

## URL Scheme {#url-scheme}

Surge 支持 URL Scheme 用于外部调用。

### 通用格式

```
surge://[action]/[parameters]
```

### 支持的 Action

- `surge://install-config?url=` — 安装配置文件
- `surge://install-module?url=` — 安装模块

## Surge Mac CLI {#surge-cli}

Surge Mac 提供命令行工具，位于 `/Applications/Surge.app/Contents/Applications/surge-cli`。

### 常用命令

```bash
# 重新加载配置
surge-cli reload

# 切换配置
surge-cli switch-profile <配置名称>

# 关闭 Surge
surge-cli stop

# 测试所有代理
surge-cli test-all-policies

# 测试指定策略组
surge-cli test-group <组名>

# 显示活动连接
surge-cli dump active

# 刷新 DNS 缓存
surge-cli flush dns

# 修改日志级别
surge-cli set-log-level verbose

# 运行诊断
surge-cli diagnostics
```

### 参数

- `--raw` — 以原始 JSON 格式输出
- `--remote/-r` — 连接到远程 Surge 实例，如 `-r password@192.168.2.2:6170`

## 托管配置 (Managed Profile) {#managed-profile}

托管配置允许配置文件由远程服务器管理，用户界面中的某些设置可被锁定，防止最终用户修改。

配置示例：

```ini
[Managed]
# 锁定 General 段落中的特定选项
lock = General > dns-server
lock = General > skip-proxy

# 隐藏特定界面元素
hide = Proxy Group > Auto
```

## HTTP API {#http-api}

Surge 提供 HTTP API 用于远程控制。

### 端点

| 端点 | 说明 |
|------|------|
| `GET /v1/policies` | 获取所有策略信息 |
| `GET /v1/policies/[policy-name]` | 获取指定策略信息 |
| `GET /v1/profiles` | 获取配置文件信息 |
| `GET /v1/rules` | 获取规则信息 |
| `GET /v1/requests/recent` | 获取最近的请求 |
| `GET /v1/requests/active` | 获取活动请求 |
| `POST /v1/requests/kill` | 终止活动请求 |

### 示例

```bash
curl -X GET "http://key@127.0.0.1:6166/v1/policies"
```
