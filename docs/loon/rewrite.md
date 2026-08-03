# 5. 复写

> 以下搬运至 [Loon官方文档 ](https://nsloon.app/docs/Rewrite/rewrite_v2)，不定时更新

复写是专门用来处理 HTTP/HTTPS 类型的请求，在请求未发出前，根据所设定的复写类型来修改请求数据，目前可修改 URL、Header 和 Body，也可以直接返回重定向、拒绝响应或 Mock 数据。所有的复写仅针对**http请求**或者**经过解密后的https请求**，并在规则匹配前执行。

![5](Photo/5.webp){: width=900}

<!-- prettier-ignore -->
!!! 提示
    Loon 3.5.1 (978) 起支持**新版 Rewrite 语法**，可以使用 [Rewrite 配置生成器](https://nsloon.app/rewrite-builder) 组合条件和 Action。本文 5.1 及以后的内容为**旧版语法**，仅用于维护旧配置，不再扩展，新配置请使用新版语法。

<!-- prettier-ignore -->
!!! 注意
    以下主要讲的是 `[Rewrite]` 区块下的内容，所以示例都以 `[Rewrite]` 开头表明在其之下，并不是让你每个参数字段前都加上 `[Rewrite]`。
    
    复写的处理会在规则匹配之前

### 5.0 新版 Rewrite 语法（Loon 3.5.1 (978)+）

#### 基本格式

每条 Rewrite 使用一行配置，基本格式为：

```text
<phase> if <condition> then <action> [| <action> ...]
```

为请求设置 Header：

```
[Rewrite]
request if ${url} ~= /^https:\/\/api\.example\.com/ then request.header.set("X-Loon", "true")
```

修改 JSON 响应：

```
[Rewrite]
response if ${url} ~= /^https:\/\/api\.example\.com\/profile$/ && ${response.status} == 200 then response.json.replace("data.vip", true)
```

多个 Action 使用 `|` 连接，并按照从左到右的顺序执行：

```
[Rewrite]
request if ${url} ~= /^https:\/\/api\.example\.com/ then request.header.set("X-Loon", "true") | request.header.del("Cookie")
```

每条 Rewrite 必须写在一行中。一条普通 Rewrite 不能同时包含请求 Action 和响应 Action。

#### 执行阶段

| 阶段 | 执行时机 | 可用数据 |
|---|---|---|
| `request` | 请求发出前 | URL、请求方法、请求 Header |
| `response` | 收到响应 Header 后 | 请求数据、响应状态码、响应 Header |

`response.body.mock(...)` 是特殊情况：配置阶段仍写作 `response`，但 Loon 会在请求发往上游前提前生成响应。

#### 条件表达式

| 操作符 | 说明 |
|---|---|
| `==` | 精确比较完整值 |
| `~=` | 使用正则查找匹配（默认查找能匹配的部分，需要完整匹配请显式使用 `^` 和 `$`） |
| `&&` / `\|\|` / `()` | 并且 / 或者 / 调整或保留条件分组 |

优先级为：**比较操作符 > `&&` > `\|\|`**。同时使用 `&&` 和 `||` 时建议使用括号。

所有动态值统一使用 `${...}`：

| 变量 | 类型 | `request` | `response` |
|---|---|---:|---:|
| `${url}` | String | ✓ | ✓ |
| `${request.method}` | String | ✓ | ✓ |
| `${request.header['name']}` | String 或 null | ✓ | ✓ |
| `${response.status}` | Number | — | ✓ |
| `${response.header['name']}` | String 或 null | — | ✓ |

Header 名称查找不区分大小写。`request` 阶段不能引用尚未生成的响应变量。当前版本不支持在 `if` 条件中读取请求或响应 Body。

##### 条件正则捕获

在正则条件后使用 `as <name>` 保存匹配结果，`${item.0}` 为完整匹配、`${item.1}` 为第一个捕获组：

```
[Rewrite]
request if ${url} ~= /^https:\/\/api\.shop\.com\/item\/(\d+)/ as item then request.header.set("X-Item-ID", "${item.1}")
```

限制：捕获名称在同一条 Rewrite 中必须唯一；不能与插件参数重名；捕获下标不能超过正则捕获组数量；被 Action 引用的捕获条件必须经过表达式的所有成功路径（不能位于 `||` 的可选分支中）。

条件正则与 Action 自带正则使用两套捕获语法：

| 位置 | 写法 | 引用 |
|---|---|---|
| `if` 条件正则 | `~= /.../ as item` | `${item.0}`、`${item.1}` |
| Header/Body Replace 正则 | Action 的 Regex 参数 | `$0`、`$1` |

`$n` 不是通用变量，不能跨 Action 使用。

##### 插件参数

插件参数在 `[Argument]` 中声明，Rewrite 中通过 `${参数名}` 引用（见[插件](plugin.md)）。`input`/`select` 需返回数字时使用 `type=number`：

```
[Argument]
enabled = switch,true,tag=启用
price = input,9.99,type=number,tag=价格
region = select,"CN","US","JP",tag=地区
```

#### Action（位置参数）

所有 Action 统一使用位置参数，按方法声明中的顺序填写，不允许填写参数名称：

```text
action(value, value)
```

##### Action 方法速查

```text
url.replace(String)
redirect(Number, String)
reject(Number[, String])
reject_img(Number)
reject_dict(Number)
reject_array(Number)
reject_video(Number)

request.header.add(String, String)
request.header.set(String, String)
request.header.del(String)
request.header.replace(String, Regex, RegexReplacement)
response.header.add/set/del/replace（同上）

request.body.replace(Regex, RegexReplacement)
response.body.replace(Regex, RegexReplacement)

request.json.add(String, Any)
request.json.delete(String)
request.json.replace(String, Any)
request.json.jq(String)
request.json.jq_file(String)
response.json.add/delete/replace/jq/jq_file（同上）

request.body.mock(String, String[, Boolean])
request.body.mock_file(String, String[, Boolean])
response.body.mock(String, String[, Number[, Boolean]])
response.body.mock_file(String, String[, Number[, Boolean]])
```

##### 示例

```text
# URL 替换（复用 if 中的 URL 正则，Action 只填替换内容）
url.replace("https://api.example.com")

# 重定向
redirect(302, "https://new.example.com")

# 拒绝
reject(404)
reject_img(200)
reject_dict(200)
reject_array(200)
reject_video(200)

# 请求/响应 Header
request.header.add("X-Loon", "true")
request.header.set("User-Agent", "Loon")
request.header.del("Cookie")
request.header.replace("User-Agent", /iPhone OS \d+/, "iPhone OS 18")

# JSON Body
response.json.replace("data.vip", true)
response.json.jq(".data.ads = []")

# Mock 响应（response.body.mock 配置阶段写作 response）
response.body.mock("json", `{"code":0,"message":"ok"}`, 200)
response.body.mock_file("json", "response_body.json", 200)
```

值类型：String 用双引号（`"hello"`），Number（`200`、`9.99`）、Boolean（`true`/`false`）、Null（`null`）、正则（`/pattern/i`）。双引号字符串支持 `${...}` 变量展开与转义；原始字符串使用反引号，不处理转义也不展开变量。

新版语法不会按空格拆分整行，因此不需要使用 `\x20`。

开发阶段曾使用过但未正式发布的 `http-request`、`http-response` 阶段名和命名参数写法不属于兼容范围。

<!-- prettier-ignore -->
!!! 提示
    以下 **5.1 及以后** 为 **旧版语法**（Loon 3.5.1 (978) 之前），仅用于维护旧配置，不再扩展。新配置请使用本文 5.0 的新版语法。

### 5.1 URL 类型复写

此类复写会修改请求的URL

- `header`：修改请求头，客户端不会感知到重定向

```
[Rewrite]
^http://www\.google\.cn header http://www.google.com
```

### 5.2 直接响应类复写

此类复写直接返回一个code位30x的重定向response

- `302`：返回一个302响应

```
[Rewrite]
^http://example.com 302 https://example.com
```

- `307`:返回一个307响应

```
[Rewrite]
^http://example.com 307 https://example.com
```

### 5.3 reject 类型

- `reject`: 直接断开连接
- `reject-200`: 返回一个200响应，响应体内容为空
- `reject_img`: 返回一个200响应，响应体内容一像素的gif
- `reject_dict`: 返回一个200响应，响应体内容为`"{}"`的空json对象字符串
- `reject_array`: 返回一个200响应，响应体内容为`"[]"`的空json数组字符串
- `reject_video`: 返回一个200响应，响应体内容为空白视频

```
[Rewrite]
^http://example.com reject
^http://example.com reject-200
^http://example.com reject-img
^http://example.com reject-dict
^http://example.com reject-array
^http://example.com reject-video
```

### 5.4 Header 类型复写

此类复写会修改请求的Header

- `header-replace`：替换 header 中指定的字段
- `header-add`：在 header 中添加一組字段
- `header-del`:删除 header 中指定的字段
- `header-replace-regex`:替换 header 中正则匹配到的字段

```
[Rewrite]
^http://example.com header-add Connection keep-alive
^http://example.com header-del Cookie
^http://example.com header-replace User-Agent Unknown
^http://example.com header-replace-regex Cookie regex Unknown
```