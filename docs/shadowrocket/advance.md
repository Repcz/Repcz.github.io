# 高级功能

> 本文档基于 [LOWERTOP/Shadowrocket](https://github.com/LOWERTOP/Shadowrocket/blob/main/README.md) 整理

## 代理分组

代理分组（策略组）是一个容器，包含多个代理节点或其他代理分组，根据规则实现灵活的路由分发。

**路径**：`配置 ⓘ > 代理分组 > 右上角 ➕`

- 填写名称，选择[类型](#代理分组类型)，通过正则匹配或手动添加策略
- 创建的代理分组只有放到[规则](rule.md)中才能发挥作用
- 支持嵌套其他代理分组形成层级结构
- 快捷进入：首页下拉 / 首页本地节点上方指示牌图标（≥ 2.2.65）

### 代理分组类型

| 类型 | 说明 |
|------|------|
| `select` | 手动选择策略 |
| `url-test` | 自动测试并选择延迟最低节点 |
| `fallback` | 节点不可用时自动切换可用节点 |
| `load-balance` | 负载均衡，相同域名使用同节点 |
| `random` | 随机分配，不区分域名 |

### 参数设置

| 参数 | 说明 |
|------|------|
| `interval` | 测试周期（秒） |
| `timeout` | 超时时间（秒） |
| `tolerance` | 公差，新节点延迟必须优于旧节点 + 公差才切换 |
| `url` | 测试地址 |
| `hidden` | 隐藏分组（见下文） |

<!-- prettier-ignore -->
!!! warning "除 `select` 类型外，不建议在自动测试分组中嵌套其他代理分组，以免策略逻辑复杂化"

### 隐藏代理分组

≥ 2.2.66 版本支持，纯文本设置，无 UI 界面：

```ini
香港节点 = url-test,HK 01,HK 02,HK 03,policy-select-name=HK 01,interval=600,tolerance=100,timeout=5,url=http://www.gstatic.com/generate_204,hidden=1
```

- `hidden=1`：隐藏
- `hidden=0`：不隐藏（默认）

### 订阅过滤与代理分组

代理分组内可开启**订阅**开关，选择所需订阅后，策略区域自动显示该订阅所有节点。支持使用[正则表达式](node.md#订阅节点筛选)筛选节点范围。

### 快速测试

代理分组右上角**测试**按钮可对分组内节点进行延迟测试。开启**测试并选择最快服务器**后，手动测试将自动选择最低延迟节点（≥ 2.2.65）。

---

## 负载均衡

`load-balance` 和 `random` 类型用于将流量平均分配给多个节点。

- **load-balance**：相同域名使用同一节点
- **random**：完全随机分配

**配置**：`配置 ⓘ > 代理分组 > 右上角 ➕`，类型选择 `load-balance` 或 `random`

---

## Hosts

自定义域名解析结果或指定 DNS 服务器。

- **UI 编辑**：`配置 ⓘ > Hosts`
- **文本编辑**：配置文件 `[Host]` 字段

```ini
[Host]
# 域名映射到指定 IP
example.com = 1.2.3.4

# 域名指定 DNS 服务器
example.com = server:1.2.3.4
```

默认代理类 DNS 在远端执行。可使用 `use-local-host-item-for-proxy` [隐式参数](config.md#隐式参数) 使本地 Host 映射对代理生效。

---

## URL 重写

修改 HTTP(S) 请求/响应 URL。

- **UI 编辑**：`配置 ⓘ > URL重写`
- **文本编辑**：配置文件 `[URL Rewrite]` 字段

支持类型：`HEADER`、`302`、`307` 及其他代理策略。

支持 `always-reject-url-rewrite` [隐式参数](config.md#隐式参数) 使 REJECT 策略在所有路由模式下生效。

---

## 头部重写

请求头/响应头重写，包含 `del`、`add`、`replace`、`replace-regex` 操作。

- **UI 编辑**：`配置 ⓘ > 头部重写`
- **文本编辑**：配置文件 `[Header Rewrite]` 字段

**类型**：

| 类型 | 说明 |
|------|------|
| `http-request` | 作用于请求头 |
| `http-response` | 作用于响应头 |

**行为**：

| 行为 | 说明 |
|------|------|
| `header-del` | 删除指定字段 |
| `header-add` | 添加字段及值 |
| `header-replace` | 替换字段值 |
| `header-replace-regex` | 正则替换字段值 |

**常用字段**：`Content-Type`、`User-Agent`、`Accept-Language`、`Host`、`Referer`、`Cookie`、`X-Forwarded-For`、`Location`

---

## 正文重写

请求体/响应体重写，支持 `jq` 语法。

- **UI 编辑**：`配置 ⓘ > 正文重写`
- **文本编辑**：配置文件 `[Body Rewrite]` 字段

**类型**：

| 类型 | 说明 |
|------|------|
| `http-request` | 请求体 |
| `http-response` | 响应体 |
| `http-request-jq` | jq 语法处理请求体 |
| `http-response-jq` | jq 语法处理响应体 |

---

## 本地映射

将请求映射到本地数据或文件。

- **UI 编辑**：`配置 ⓘ > 本地映射`
- **文本编辑**：配置文件 `[Map Local]` 字段

**类型**：

| 类型 | 说明 |
|------|------|
| `text` | 返回文本类型响应 |
| `file` | 返回文件类型响应（支持远程文件地址） |
| `tiny-gif` | 返回 1x1 像素 GIF |
| `base64` | 返回 Base64 编码数据 |

---

## HTTPS 解密

对启用解密的配置文件生效，使包含 MITM 的模块完整生效。

### 开启方法

1. `配置 ⓘ > HTTPS 解密 > 证书 > 生成新的 CA 证书 > 安装证书`
2. `系统设置 > 已下载描述文件 > 安装`
3. `系统设置 > 通用 > 关于本机 > 证书信任设置 > 开启对应证书信任`

### HTTP/2 MitM

≥ 2.2.81 版本支持通过 HTTP/2 进行 MITM：

- UI：`配置 ⓘ > HTTPS 解密 > 通过 HTTP/2 进行中间人攻击（MitM）`
- 纯文本：`[MITM]` 字段添加 `h2 = true`

模块中的此命令会覆盖配置文件设置。

<!-- prettier-ignore -->
!!! warning "切换配置文件需要重新开启解密。可使用[证书模块](#证书模块)避免重复安装"

### 身份证书密码

安装 CA 证书时提示输入「身份证书的密码」，可尝试输入：`Shadowrocket`

---

## 证书模块

免除切换配置文件时重复安装 CA 证书。模块优先级高于配置文件，使解密状态不受更换配置影响。

### 创建证书模块

1. `配置 > 模块 > 新建模块`，粘贴以下内容：
```ini
#!name=证书
[MITM]
enable=true

# 下方粘贴证书密码
ca-passphrase=

# 下方粘贴证书内容
ca-p12=
```

2. 点击已安装证书的配置文件 `ⓘ > HTTPS 解密 > 证书 ⓘ > 复制`
3. 将复制的证书内容粘贴至 `ca-p12=` 后保存

<!-- prettier-ignore -->
!!! warning "请确保完成解密证书安装并信任后再使用证书模块，否则显示 `No PKCS12 Certificates`"

---

## 脚本

通过 JavaScript 实现高级处理逻辑。

- **UI 编辑**：`配置 ⓘ > 脚本`
- **文本编辑**：配置文件 `[Script]` 字段

### 脚本类型

| 类型 | 说明 |
|------|------|
| `http-request` | 修改 HTTP 请求 |
| `http-response` | 修改 HTTP 响应 |
| `event` | 事件触发 |
| `rule` | 脚本规则 |
| `dns` | DNS 解析器 |
| `cron` | 定时运行 |

### 脚本引擎

| 引擎 | 说明 |
|------|------|
| `auto` | 自动选择 |
| `jsc` | 适合小型简单脚本 |
| `webview` | 适合复杂高内存脚本 |

> `http-request`/`http-response` 类型默认调用 WebView 引擎，建议明确指定

### 示例

```ini
[Script]
script = type=rule,script-path=rule.js
```

---

## 模块

模块是 Shadowrocket 的插件/扩展，写法与[配置文件](config.md)相同。模块的[规则优先级](rule.md#规则优先级)高于配置文件。

### 下载远程模块

`配置 > 模块 > 右上角 ➕ > 填写链接 > 下载`

### 本地新建模块

`配置 > 模块 > 新建模块 > 编辑后保存`

模块 `[MITM]` 部分需加 `%APPEND%`，避免覆盖配置内容：

```ini
[MITM]
hostname = %APPEND% 主机名
```

### 模块管理

`配置 > 模块 > 右上角 •••`

可排序、批量删除/启用/禁用模块。

### 注意事项

- 包含 MITM 的模块需[开启 HTTPS 解密](advance.md#https-解密)或使用[证书模块](advance.md#证书模块)
- 不含规则类的模块在非配置模式下也可生效
- iOS 15 以下系统可能因内存不足导致[模块失效](faq.md#模块失效)或 [VPN 断开](faq.md#vpn-自动断开)

---

## 编辑参数

`配置 > 模块 > 点击模块 > 编辑参数`

用于动态修改模块中的参数，配合 `URL Rewrite`、`Script`、`MITM` 等使用。修改内容不因模块更新而丢失。

示例模块参数定义：

```ini
#!name=证书模块
#!arguments=证书模块:false, 证书内容:此处粘贴有效的证书内容, 证书密码:此处粘贴有效的证书密码
#!arguments-desc=证书模块：启用并填写证书内容方可生效\n\n证书内容：粘贴自有效证书\n\n证书密码：若默认密码是Shadowrocket则可不动

[MITM]
enable={{{证书模块}}}
ca-passphrase={{{证书密码}}}
ca-p12={{{证书内容}}}
```

---

## 脚本 URL

`配置 > 配置文件 > 编辑配置 > 脚本 URL`

当前使用的所有远程脚本资源列表：

- ✅ 下载成功
- ❎ 下载失败（可点击重新拉取）
- 点击[使用配置](config.md#使用配置)/[编译配置](config.md#编译配置)更新所有 URL

---

> 继续阅读：[**常见问题与附录** &rarr;](faq.md)
