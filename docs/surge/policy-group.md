# 策略组

策略组用于将多个代理策略组合在一起，通过不同的选择策略决定最终使用的代理。策略组在 `[Proxy Group]` 段落中定义。

## 手动选择组 (Select)

在用户界面上手动选择要使用的策略：

```ini
[Proxy Group]
SelectGroup = select, ProxyHTTP, ProxyHTTPS, DIRECT, REJECT
```

- Surge iOS：使用小组件快速切换策略
- Surge Mac：在菜单栏菜单中切换策略

## 自动测试组 (URL-Test)

通过测试到目标 URL 的延迟，自动选择最优策略：

```ini
[Proxy Group]
AutoTestGroup = url-test, ProxySOCKS5, ProxySOCKS5TLS
```

### 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `url` | URL | `http://www.gstatic.com/generate_204` | 测试 URL |
| `interval` | 秒 | 600 | 测试结果的有效期 |
| `tolerance` | 毫秒 | 100 | 切换容差，防止频繁切换 |
| `timeout` | 秒 | 5 | 测试超时 |
| `evaluate-before-use` | 布尔 | false | 首次使用前等待测试完成 |

示例：

```ini
Auto = url-test, ProxyA, ProxyB, ProxyC, url=http://www.gstatic.com/generate_204, interval=600, tolerance=50, timeout=5, evaluate-before-use=true
```

### 临时覆盖

手动选择策略可临时覆盖自动测试结果：

- Surge Mac：主菜单中对应的组里找到覆盖选项
- Surge iOS：策略组视图中长按对应策略的菜单找到覆盖选项

## 降级组 (Fallback)

根据优先级和可用性选择策略。定义在前面的策略具有更高优先级：

```ini
[Proxy Group]
FallbackGroup = fallback, ProxySOCKS5, ProxySOCKS5TLS
```

### 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `url` | URL | 默认测试 URL | 测试 URL |
| `interval` | 秒 | 600 | 测试结果有效期 |
| `timeout` | 秒 | 5 | 测试超时 |

### 临时覆盖

同自动测试组，可通过手动选择临时覆盖。

## 负载均衡组 (Load Balance)

从可用的子策略中随机选择一个策略：

```ini
[Proxy Group]
LoadBalanceGroup = load-balance, ProxyA, ProxyB, ProxyC
```

### 参数

| 参数 | 说明 |
|------|------|
| `persistent=true` | 对相同的目标主机名使用相同策略，避免触发风控 |

```ini
LB = load-balance, ProxyA, ProxyB, persistent=true
```

## 子网组 (Subnet)

根据当前网络环境自动选择策略。使用子网表达式作为条件：

```ini
[Proxy Group]
SubnetGroup = subnet, default=ProxyHTTP, TYPE:WIFI=ProxyHTTP, SSID:MyHome=ProxySOCKS5
```

### 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `default` | 是 | 无匹配时的默认策略 |
| `TYPE:WIFI` | 否 | 匹配所有 Wi-Fi 网络 |
| `TYPE:WIRED` | 否 | 匹配有线网络 |
| `TYPE:CELLULAR` | 否 | 匹配蜂窝网络 |
| `SSID:name` | 否 | 匹配指定 SSID |
| `BSSID:mac` | 否 | 匹配指定 BSSID |

!!! note "兼容性"
    从 Surge iOS 4.12.0 / Surge Mac 4.5.0 起，SSID 组更名为子网组。依然支持旧语法 `ssid` 作为组类型关键字。

## 包含策略 (Policy Including)

### 从外部文件/URL 导入

```ini
[Proxy Group]
egroup = select, policy-path=https://example.com/proxies.txt, update-interval=86400
```

策略文件内容：

```ini
Proxy-A = https, example1.com, 443
Proxy-B = https, example2.com, 443
```

#### 参数

| 参数 | 说明 |
|------|------|
| `policy-path` | 外部策略列表的 URL 或文件路径 |
| `update-interval` | 更新间隔（秒），仅 URL 有效 |
| `policy-regex-filter` | 正则表达式过滤策略名称 |
| `external-policy-modifier` | 修改外部策略参数，如 `test-url=http://apple.com/,tfo=true` |
| `external-policy-name-prefix` | 为外部策略名增加前缀 |

### 包含现有策略

```ini
# 包含 [Proxy] 中定义的所有代理
GroupA = select, include-all-proxies=true

# 包含所有代理并过滤
GroupB = select, include-all-proxies=true, policy-regex-filter=^(?!.*REJECT).*$

# 包含其他策略组的策略
GroupC = select, include-other-group="GroupA,GroupB"
```

## 通用策略组参数

| 参数 | 说明 |
|------|------|
| `no-alert` | 不显示策略变更通知 |
| `hidden` | 不在菜单和策略选择视图中显示 |

```ini
HiddenGroup = url-test, ProxyA, ProxyB, hidden=true, no-alert=true
```
