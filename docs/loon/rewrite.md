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

```text
<phase> if <condition> then <action> [| <action> ...]
```

为请求添加 Header：

```
[Rewrite]
http-request if ${url} ~= /^https:\/\/api\.example\.com/ then request.header.set(name="X-Loon", value="true")
```

修改 JSON 响应：

```
[Rewrite]
http-response if ${url} ~= /^https:\/\/api\.example\.com\/profile$/ && ${response.status} == 200 then response.json.replace(path="data.vip", value=true)
```

多个 Action 使用 `|` 连接，并从左到右执行：

```
[Rewrite]
http-request if ${url} ~= /^https:\/\/api\.example\.com/ then request.header.set(name="X-Loon", value="true") | request.header.delete(name="Cookie")
```

每条 Rewrite 必须写在一行中。

#### 执行阶段

| 阶段 | 时机 | 可用数据 |
|---|---|---|
| `http-request` | 请求发出前 | URL、请求方法、请求 Header |
| `http-response` | 收到响应 Header 后 | 请求数据、响应状态码、响应 Header |

#### 条件与变量

| 操作符 | 说明 |
|---|---|
| `==` | 精确相等 |
| `~=` | 正则匹配（默认查找能匹配的部分，需要完整匹配请使用 `^` 和 `$`） |
| `&&` / `\|\|` / `()` | 逻辑与 / 或 / 调整优先级 |

所有动态值都使用 `${...}`：

| 变量 | 类型 | 请求阶段 | 响应阶段 |
|---|---|---:|---:|
| `${url}` | String | ✓ | ✓ |
| `${request.method}` | String | ✓ | ✓ |
| `${request.header['name']}` | String 或 null | ✓ | ✓ |
| `${response.status}` | Number | — | ✓ |
| `${response.header['name']}` | String 或 null | — | ✓ |

Header 名称不区分大小写。请求阶段不能引用响应变量，当前版本也不支持在条件中读取请求或响应 Body。

正则捕获使用 `as <name>` 保存匹配结果，`${item.0}` 为完整匹配、`${item.1}` 为第一个捕获组：

```
[Rewrite]
http-request if ${url} ~= /^https:\/\/api\.shop\.com\/item\/(\d+)/ as item then request.header.set(name="X-Item-ID", value="${item.1}")
```

插件参数在 `[Argument]` 中声明，Rewrite 中通过 `${参数名}` 引用（见[插件](plugin.md)）。

#### Action 示例

```text
# URL 替换
url.replace(pattern=/^http:\/\/example\.com/, replacement="https://api.example.com")

# 重定向
redirect(status=302, location="https://new.example.com")

# 拒绝
reject(status=200, body="json-object")

# 请求/响应 Header
request.header.add(name="X-Loon", value="true")
request.header.set(name="User-Agent", value="Loon")
request.header.delete(name="Cookie")
request.header.replace(name="User-Agent", pattern=/iPhone OS \d+/, replacement="iPhone OS 18")

# JSON Body
response.json.replace(path="data.vip", value=true)

# Mock 响应
response.body.mock(type="json", data=`{"code":0,"message":"ok"}`, status=200)
```

值类型：String 用双引号（`"hello"`），Number（`200`、`9.99`）、Boolean（`true`/`false`）、Null（`null`）、正则（`/pattern/i`）。双引号字符串支持 `${...}` 变量展开与转义；原始字符串使用反引号，不处理转义也不展开变量。

新版语法不会按空格拆分整行，因此不需要使用 `\x20`。

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