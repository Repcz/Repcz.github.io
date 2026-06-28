# 规则系统

> 本文档基于 [LOWERTOP/Shadowrocket](https://github.com/LOWERTOP/Shadowrocket/blob/main/README.md) 整理

规则用来决定哪些网络请求走代理、哪些直连。软件对规则集的识别不受文件后缀影响，以内容为识别标准。

## 添加规则

- **手动添加**：`配置 ⓘ > 规则 > 右上角 ➕`，选择[规则类型](#规则类型)和[规则策略](#规则策略)
- **从日志添加**：`数据 > 代理 > 启用日志记录` > 点击记录 > `•••` > 选择类型添加规则

### 规则管理

`配置 > 配置文件 > 编辑配置 > 规则 > 右上角 •••`

可选择并批量删除规则。

---

## 规则优先级

规则从上到下逐条匹配，匹配后停止继续匹配：

1. **模块中的规则**优先于配置文件中的规则
2. **上面的规则**优先于下面的规则
3. **域名类规则**优先于 IP 类规则

自动编译时遵循：**显式优先 > 推断其次 > 最终兜底**

- 显式规则（`DOMAIN`、`DOMAIN-SUFFIX`、`URL-REGEX` 等）优先级最高
- 推断规则（`GEOIP` 等）次之
- `FINAL` 作为兜底始终在末尾

---

## 规则类型

| 类型 | 说明 | 示例 |
|------|------|------|
| `DOMAIN-SUFFIX` | 匹配域名后缀 | `DOMAIN-SUFFIX,example.com,DIRECT` 匹配 `a.example.com` |
| `DOMAIN-KEYWORD` | 匹配域名关键词 | `DOMAIN-KEYWORD,exa,DIRECT` 匹配含 `exa` 的域名 |
| `DOMAIN-WILDCARD` | 通配符匹配 | `DOMAIN-WILDCARD,a*c.example*.com,DIRECT` |
| `DOMAIN` | 完整域名匹配 | `DOMAIN,www.example.com,DIRECT` 仅匹配 `www.example.com` |
| `USER-AGENT` | 匹配 UA（支持 `*`） | `USER-AGENT,MicroMessenger*,DIRECT` |
| `URL-REGEX` | URL 正则匹配 | `URL-REGEX,^https?://.+/item.+,REJECT` |
| `IP-CIDR` | IP 段匹配 | `IP-CIDR,192.168.1.0/24,DIRECT` |
| `IP-ASN` | ASN 编号匹配 | `IP-ASN,56040,DIRECT` |
| `RULE-SET` | 规则集匹配 | 规则集内容需包含规则类型 |
| `DOMAIN-SET` | 域名集匹配 | 域名集内容不包含规则类型 |
| `SCRIPT` | 脚本规则 | 匹配脚本名称 |
| `DST-PORT` | 目标端口匹配 | `DST-PORT,443,DIRECT` |
| `GEOIP` | IP 地理位置 | `GEOIP,CN,DIRECT` |
| `FINAL` | 兜底规则 | `FINAL,PROXY` |
| `PROTOCOL` | 传输协议匹配 | 仅可嵌套于逻辑规则中 |

### 逻辑规则

| 类型 | 说明 | 示例 |
|------|------|------|
| `AND` | 与 | `AND,((DOMAIN,www.example.com),(DST-PORT,123)),DIRECT` |
| `OR` | 或 | `OR,((DST-PORT,123),(DST-PORT,456)),DIRECT` |
| `NOT` | 非 | `NOT,((DST-PORT,123)),DIRECT` |
| `PROTOCOL` | 协议匹配 | 仅嵌套于逻辑规则：`AND,((PROTOCOL,UDP),(DST-PORT,443)),REJECT-NO-DROP` |

### IP-CIDR 与 DNS 解析

域名请求遇到 IP 类规则时，Shadowrocket 会向本地 DNS 发送查询以判断 IP 是否匹配。

- 加 `no-resolve` 则跳过 DNS 查询：`IP-CIDR,172.16.0.0/12,DIRECT,no-resolve`
- 详见 [no-resolve 的作用](dns.md#no-resolve-的作用)

---

## 规则策略

| 策略 | 说明 |
|------|------|
| `PROXY` | 代理转发 |
| `DIRECT` | 直连 |
| `TAILSCALE` | 通过 Tailscale 隧道转发 |
| `REJECT` | 拒绝，返回 HTTP 404 |
| `REJECT-DICT` | 拒绝，返回 HTTP 200 + 空 JSON 对象 `{}` |
| `REJECT-ARRAY` | 拒绝，返回 HTTP 200 + 空 JSON 数组 `[]` |
| `REJECT-200` | 拒绝，返回 HTTP 200 |
| `REJECT-IMG` | 拒绝，返回 HTTP 200 + 1px GIF |
| `REJECT-TINYGIF` | 拒绝，返回 HTTP 200 + 1px GIF |
| `REJECT-DROP` | 拒绝，丢弃 IP 包 |
| `REJECT-NO-DROP` | 拒绝，返回 ICMP 端口不可达 |

此外，策略还可以选择**分组**、**代理分组**、**订阅**、**服务器节点**等。

---

## 策略扩展参数

| 参数 | 说明 |
|------|------|
| `extended-matching` | 扩展匹配，同时匹配 SNI 和 HTTP Host 标头。Shadowrocket 默认支持 |
| `pre-matching` | 预匹配，快速低开销拒绝请求。REJECT 策略专用，在正常规则匹配前生效，最高优先级 |

纯文本示例：

```ini
[Rule]
DOMAIN,ad.com,REJECT,extended-matching,pre-matching
```

---

## APP 分流

iOS 系统无常规分应用代理操作，通过域名/IP/UA 规则实现 App 分流。

示例：YouTube App 分流走代理

1. 复制 YouTube 规则集链接：
   ```
   https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Shadowrocket/YouTube/YouTube.list
   ```
2. `配置 ⓘ > 规则 > 右上角 ➕`，类型选 `RULE-SET`，策略选 `PROXY`，粘贴链接

<!-- prettier-ignore -->
!!! tip "若引用的是域名集，规则类型选择 `DOMAIN-SET`"

---

## 规则集 URL

`配置 > 配置文件 > 编辑配置 > 规则集 URL`

当前使用的所有远程规则资源列表：

- ✅ 下载成功
- ❎ 下载失败（可点击重新拉取）
- 点击[使用配置](config.md#使用配置)/[编译配置](config.md#编译配置)可更新所有 URL 并重新编译

---

## 更新规则集

- **手动**：点击配置文件 > [使用配置](config.md#使用配置)/[编译配置](config.md#编译配置)
- **自动**：[自动更新](faq.md#自动更新)

## 预览规则集

`点击配置文件 > 编辑配置 > 规则 > 左滑规则集 > 预览`

---

## 替换策略

`配置 > 点击配置文件 > 编辑配置 > 替换策略`

- 点击查找右侧 `ⓘ` 选择需替换的策略
- 点击替换右侧 `ⓘ` 选择目标策略
- 点击完成

---

## 测试规则

- `配置 > 测试规则`
- `配置 > 点击配置文件 > 编辑配置 > 测试规则`

根据当前配置测试指定请求的匹配规则及最终策略。

---

> 继续阅读：[**DNS 设置** &rarr;](dns.md)
