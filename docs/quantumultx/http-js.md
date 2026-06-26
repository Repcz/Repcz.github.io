# 8. HTTP 请求


「HTTP 请求」所执行的功能，可简单的理解为执行脚本

脚本分为 「CRON 定时脚本任务」、「UI交互脚本」、「网络切换脚本」

其对应的配置文件位置为 `[task_local]`

可通过首页下方工具栏进入（默认UI），或　点击右下角「风车」进入设置 → 下方「工具 & 分析」- 「HTTP 请求」

![UI12-1](Photo/UI12-1.webp){: width=600}

### 8.1 添加 HTTP 请求

<!-- prettier-ignore -->
!!! 注意
    以下主要讲的是 `[task_local]` 区块下的内容，所以示例都以 `[task_local]` 开头表明在其之下，并不是让你每个参数字段前都加上 `[task_local]`。


#### 8.1.1 配置文本添加

```
[task_local]
event-network https://raw.githubusercontent.com/xream/scripts/main/surge/modules/network-info/net-lsp-x.js, tag=网络信息变化 𝕏, img-url=https://raw.githubusercontent.com/Koolson/Qure/master/IconSet/Color/Global.webp, enabled=true
```

```
[task_local]
0 0 1 1 * https://github.com/sub-store-org/Sub-Store/releases/latest/download/cron-sync-artifacts.min.js, tag=SubStore同步, img-url=https://raw.githubusercontent.com/fmz200/wool_scripts/main/icons/apps/SubStore.webp, enabled=true
```

```
[task_local]
event-interaction https://raw.githubusercontent.com/xream/scripts/main/surge/modules/network-info/net-lsp-x.js, tag=网络信息 𝕏, img-url=https://raw.githubusercontent.com/Koolson/Qure/master/IconSet/Color/Global.webp, enabled=true
```

<!-- prettier-ignore -->
!!! 提示
    就像任何浏览器一样，脚本 URL 字符串中 `#` 之后的内容永远不会发送到服务器。可以在脚本 URL 后添加 `#` 来附加自定义参数，并在脚本中使用 API `$environment.sourcePath` 获取完整路径并提取自定义参数。例如：
    ```
    * * * * * https://example.com/sample.js#this-is-not-sent-to-server&key1=value1&key2=value2, tag=Sample, enabled=true
    ```
    `$environment.sourcePath` 自 QX v1.0.25 起可用，支持 task 和 rewrite 两种脚本。

其基本格式为

```
<定时/事件> <脚本URL>#<自定义参数>, tag=<资源标签>, img-url=<图标URL>, require-devices=<设备ID>, enabled=<是否启用>
```

所有参数均为 `key=value` 格式（除最前面的脚本参数和 URL 外），顺序不限。

脚本参数:

  - `event-network`：对应「网络切换脚本」，当网络发生变化时自动执行（相关任务仅在 Quantumult X Tunnel 运行时触发）

  - `event-interaction`：对应「UI交互脚本」，须与UI交互执行（相关任务仅在 Quantumult X Tunnel 运行时触发）

  - `5或6位CRON表达式`：对应「CRON 定时脚本任务」，5位表达式从分开始，支持 5 或 6 位 CRON（不含 command 字段），具体请自行谷歌

资源链接：脚本资源位置，可为远程脚本，也可为本地脚本（保存在「我的 iPhone - Quantumult X - Scripts」或「iCloud Drive - Quantumult X - Scripts」）

`tag`：资源标签，可选

`img-url`：资源图标，可选

`require-devices`：可选参数，指定仅在特定设备ID上加载此任务，多个设备用逗号分隔（设备ID可在「设置 - 其他设置 - 关于」中查找）

`enabled`：是否启用该 HTTP 请求，若不使用可改为 `false`；

<!-- prettier-ignore -->
!!! 注意
    脚本任务需 Quantumult X Tunnel （VPN） 处于运行状态，以及计划任务开关(右上角⏰)为开启状态
    
    默认 HTTP 请求超时时间为 10 秒。

### 8.2 `$task.fetch()` API

`$task.fetch()` 可组装 HTTP 请求并处理响应，仅支持文本正文。一个 `$task.fetch()` 可嵌入另一个 `$task.fetch()` 的完成处理器中（用于实现串行请求而非并发请求）。

```
$task.fetch(myRequest).then(response => {
    // response.statusCode, response.headers, response.body
    $done();
}, reason => {
    // reason.error
    $done();
});
```

**请求参数 `myRequest` 格式：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `url` | string | 请求 URL，必填 |
| `method` | string | HTTP 方法，可选，默认 GET |
| `headers` | object | 请求头，可选 |
| `body` | string | 请求体，可选 |
| `opts.redirection` | bool | 是否自动跟随重定向，可选，默认 true |
| `opts.policy` | string | 指定策略发送请求，如 `"direct"`，可选。注意：每次请求都会独立完成 TCP + TLS 握手，不会复用连接 |

**响应 `response` 对象：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `statusCode` | number | HTTP 状态码 |
| `headers` | object | 响应头 |
| `body` | string | 响应体（文本） |

#### 8.1.2 UI 添加

![UI12-2](Photo/UI12-2.webp)

- 如果给的是「完整的任务格式」，如：
  
```
event-network https://raw.githubusercontent.com/xream/scripts/main/surge/modules/network-info/net-lsp-x.js, tag=网络信息变化 𝕏, img-url=https://raw.githubusercontent.com/Koolson/Qure/master/IconSet/Color/Global.webp, enabled=true
```

  可通过右上角 `＋`→ `文本方式添加`，直接粘贴即可 

![UI12-3](Photo/UI12-3.webp){: width=600}

- 如果给的是「定时任务类型」，且为单独的脚本链接(或本地脚本文件)，请阅读脚本内容，根据其需要，设置 CRON表达式

  可通过右上角 `＋`→ `高级`，填写参数即可 

![UI12-4](Photo/UI12-4.webp){: width=600}

#### 8.2 执行脚本

<!-- prettier-ignore -->
!!! 注意
    脚本任务需 Quantumult X Tunnel （VPN） 处于运行状态，以及计划任务开关(右上角⏰)为开启状态

- `event-network`：对应「网络切换脚本」，当网络发生变化时自动执行


- `event-interaction`：对应「UI交互脚本」，须与UI交互执行

  启动 VPN 后，长按节点，即可执行

![UI12-5](Photo/UI12-5.webp){: width=600}


- `5或6位CRON表达式`：对应「CRON 定时脚本任务」，5位表达式从分开始，具体请自行谷歌。也可左滑点击按钮执行或查看执行时的日志

#### 8.3 任务仓库

- 添加任务仓库

<details>
  <summary> 路径需要填写资源的raw格式链接，点此查看教程</summary>

以下方的链接举例(这是个网页，不是真正能使用的资源链接)：

```
https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/QuantumultX/12306/12306.list
```

例如在末尾添加`?raw=true`：

```
https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/QuantumultX/12306/12306.list?raw=true
```

或者直接点击 `raw` 或 `view`，使用跳转后的链接：

```
https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/QuantumultX/12306/12306.list
```

![raw1](Photo/raw1.webp)

![raw2](Photo/raw2.webp)



或者将链接里的 `blob` 修改为 `raw`：

```
https://github.com/blackmatrix7/ios_rule_script/raw/master/rule/QuantumultX/12306/12306.list
```



</details>


![UI12-6](Photo/UI12-6.webp){: width=600}

- 从仓库中添加任务

![UI12-7](Photo/UI12-7.webp){: width=600}

### 8.4 本地 HTTP 后端 `[http_backend]`

<!-- prettier-ignore -->
!!! 提示
    Quantumult X 支持部署本地 HTTP 服务器并使用 JavaScript 进行数据处理。部署后可通过以下地址访问：
    - `http://127.0.0.1:9999/your-path/your-api/`
    - `http://quantumult-x:9999/your-path/your-api/`
    - `http://localhost:9999/your-path/your-api/`
    
    由于 Network Extension 存在内存限制，请尽量保持后端脚本精简。

```
[http_backend]
;https://raw.githubusercontent.com/crossutility/Quantumult-X/master/sample-backend.js, tag=fileConverter, path=^/example/v1/
;preference.js, tag=userPreference, path=^/preference/v1/, img-url=https://example.com, enabled=true
;sample.js, tag=sample, path=^/sample/v1/, require-devices=ID1, ID2, enabled=true
```

其基本格式为：

```
<脚本路径>, tag=<资源标签>, path=<匹配路径>, img-url=<图标>, require-devices=<设备ID>, enabled=<是否启用>
```

- 脚本路径：JavaScript 脚本的远程 URL 或本地文件路径
- `tag`：资源标签，必填，用于标识此后端服务
- `path`：匹配路径，使用正则表达式，如 `^/example/v1/`
- `img-url`：资源图标，可选
- `require-devices`：可选，指定仅在特定设备ID上加载此后端
- `enabled`：是否启用

**脚本可用变量：** `$request.url`、`$request.path`、`$request.headers`、`$request.body`、`$prefs`、`$task`、`$notify(title, subtitle, message)`、`console.log(message)`

**输出：** `$done({status:"HTTP/1.1 200 OK", headers:{}, body:"here is a string"})`

还可以使用签名或其他验证方法来校验请求的合法性。

完整示例见 [sample-backend.js](https://github.com/crossutility/Quantumult-X/blob/master/sample-backend.js)。

### 8.5 `$configuration.sendMessage()` API

自 QX v1.0.22+ (iOS 13.0+) 起，脚本可通过 `$configuration.sendMessage()` 与 Quantumult X 内核交互。

**基本用法：**
```
$configuration.sendMessage(message).then(resolve => {
    if (resolve.error) { console.log(resolve.error); }
    if (resolve.ret)   { console.log(JSON.stringify(resolve.ret)); }
    $done();
}, reject => {
    $done();
});
```

**支持的操作：**

| Action | 说明 | 可用版本 |
|--------|------|---------|
| `get_policy_state` | 获取所有策略组状态 | v1.0.22+ |
| `set_policy_state` | 设置静态策略组选择（仅 static 类型及内置 proxy） | v1.0.22+ |
| `url_latency_benchmark` | 对指定节点进行延迟测试 | v1.0.22+ |
| `get_customized_policy` | 获取自定义策略信息 | v1.0.22+ |
| `get_running_mode` | 获取当前运行模式 | v1.0.22+ |
| `set_running_mode` | 设置运行模式（`all_proxy` / `all_direct` / `filter`） | v1.0.22+ |
| `get_server_description` | 获取节点配置描述（仅 event-interaction 脚本） | v1.0.30+ |
| `dns_clear_cache` | 清除所有 DNS 缓存 | v1.0.22+ |
| `dns_get_placeholder_ip` | 获取域名的 placeholder IP 映射 | v1.0.22+ |
| `dns_update_cache` | 插入/更新 DNS A 记录 | v1.0.22+ |
| `get_traffic_statistics` | 获取流量统计（字节） | v1.0.28+ |

完整示例见 [sample-configuration-api.js](https://github.com/crossutility/Quantumult-X/blob/master/sample-configuration-api.js)、[sample-dns-api.js](https://github.com/crossutility/Quantumult-X/blob/master/sample-dns-api.js)、[sample-get-traffic-statistics.js](https://github.com/crossutility/Quantumult-X/blob/master/sample-get-traffic-statistics.js)。

### 8.6 `$iCloud` API

自 QX v1.0.31+ 起，脚本可读写 iCloud 文件（位于 `iCloud Drive/Quantumult X/Data/`）：

| API | 说明 |
|-----|------|
| `$iCloud.writeFile(Uint8Array, path)` | 写入文件 |
| `$iCloud.readFile(path)` | 读取文件，返回 `Uint8Array` |
| `$iCloud.removeFile(path)` | 删除文件 |

完整示例见 [sample-icloud-storage.js](https://github.com/crossutility/Quantumult-X/blob/master/sample-icloud-storage.js)。
