# 2. 节点订阅

<!-- prettier-ignore -->
!!! 注意
    以下主要讲的是 `[server_local]`、`[server_remote]` 或 `[general]` 区块下的内容，所以示例以`[server_local]`、`[server_remote]` 或 `[general]`  开头表明在其之下，并不是让你每个参数字段前都加上 `[server_local]`、`[server_remote]` 或 `[general]` 。
    
    以 `;` 或 `#` 或 `//` 开头的行为注释行。


### 2.1 添加远程节点订阅

#### 2.1.1 UI添加

- 长按最上方「节点」按钮，即可进入添加机场订阅，效果和下图是一致的
- 切记不要使用一键导入，而是复制通用订阅；如果是非通用订阅，可以尝试开启资源解析器进行使用(本地转换，无泄漏风险，但不代表转换后一定可用)
- 如果你的解析器是空的，「节点页」点击左下角编辑，需要在 `[general]` 下添加：

```
resource_parser_url=https://raw.githubusercontent.com/KOP-XIAO/QuantumultX/master/Scripts/resource-parser.js 
```

![添加机场订阅](https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/%E8%AE%A2%E9%98%85.webp) 

#### 2.1.2 配置文件添加

```
[server_remote]
https://example.com/provider.txt, tag=机场名称, img-url=hhttps://example.com/airport.webp, update-interval=86400, opt-parser=false, inserted-resource=false, enabled=true 
```

对应的完整参数：

`<资源路径>, <资源标签>, <资源图标>，<自动更新时间间隔>, <是否使用资源解析器>, <插入资源>, <是否启用>`

- `tag` 资源标签：相当于名称，标识这条节点订阅的作用；

- `img-url` 自定义图标参数是可选的，使用它可以更美观些，但此处不显示彩色图标

- `update-interval` 自动更新的时间间隔，单位为秒；

- `opt-parser` 是否使用资源解析器，若关闭则改为 `false`；

- `inserted-resource` 插入资源，将文件中的节点放置于本地节点之前；

- `enabled` 是否启用该节点订阅文件，若不使用可改为 `false`；


### 2.2 添加本地节点

#### 2.2.1 配置文件

将节点内容复制粘贴到配置文件`[server_loacl]`下,会增加一个「已保存」

但是这样做会存在一个问题：无法在策略组中使用正则参数进行筛选，因此建议使用配置片段进行添加

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/server_loacl_1.webp" width="1200">

#### 2.2.2 配置片段添加

使用配置片段的方式添加，类似于添加远程节点订阅，只不过引用的远程资源是本地文件

<!-- prettier-ignore -->
!!! 提示
    添加配置片段时，无需带有`[server_local]`、`[server_remote]`、`[filter_local]`、`[filter_remote]` 等字段

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/server_loacl_2.webp" width="900">

#### 2.2.3 节点格式

  > 搬运至[@KOP-XIAO](https://github.com/KOP-XIAO/QuantumultX/blob/4a705faac3cfed6ba0e55d6a71bc3b35036f0f96/QuantumultX_Profiles.conf#L237)


##### shadowsocks 以及 shadowsocksR 类型

支持 V2-Plugin；支持UDP，支持UDP-OVER-TCP

```
shadowsocks=ss-a.example.com:80, method=chacha20, password=pwd, obfs=http, obfs-host=bing.com, obfs-uri=/resource/file, fast-open=false, udp-relay=false, server_check_url=http://www.apple.com/generate_204, tag=Sample-A

shadowsocks=ss-b.example.com:80, method=chacha20, password=pwd, obfs=http, obfs-host=bing.com, obfs-uri=/resource/file, fast-open=false, udp-relay=false, tag=Sample-B

shadowsocks=ss-c.example.com:443, method=chacha20, password=pwd, obfs=tls, obfs-host=bing.com, fast-open=false, udp-relay=false, tag=Sample-C

shadowsocks=ssr-a.example.com:443, method=chacha20, password=pwd, ssr-protocol=auth_chain_b, ssr-protocol-param=def, obfs=tls1.2_ticket_fastauth, obfs-host=bing.com, tag=Sample-D

shadowsocks=ws-a.example.com:80, method=aes-128-gcm, password=pwd, obfs=ws, obfs-uri=/ws, fast-open=false, udp-relay=false, tag=Sample-E

shadowsocks=ws-b.example.com:80, method=aes-128-gcm, password=pwd, obfs=ws, fast-open=false, udp-relay=false, tag=Sample-F

shadowsocks=ws-tls-a.example.com:443, method=aes-128-gcm, password=pwd, obfs=wss, obfs-uri=/ws, fast-open=false, udp-relay=false, tag=Sample-G

shadowsocks=ws-tls-a.example.com:443, method=aes-128-gcm, password=pwd, udp-over-tcp=true fast-open=false, udp-relay=false, tag=Sample-H
```

##### vmess 类型

ws，wss(ws+tls)，over-tls，tcp，支持 UDP；

vmess 类型节点默认开启 aead，关闭请用 aead=false

- ws 类型

```
vmess=ws-c.example.com:80, method=chacha20-ietf-poly1305, password= 23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs-host=ws-c.example.com, obfs=ws, obfs-uri=/ws, fast-open=false, udp-relay=false, aead=false, tag=Sample-H
```

- wss(ws+tls) 类型

```
vmess=ws-tls-b.example.com:443, method=chacha20-ietf-poly1305, password= 23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs-host=ws-tls-b.example.com, obfs=wss, obfs-uri=/ws, tls-verification=true,fast-open=false, udp-relay=false, tag=Sample-I
```
  
- http 类型

```
vmess=example.com:80, method=chacha20-poly1305, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs=http, obfs-host=bing.com, obfs-uri=/resource/file, fast-open=false, udp-relay=false, server_check_url=http://www.apple.com/generate_204, tag=vmess-http
```

- tcp 类型

```
vmess=vmess-a.example.com:80, method=aes-128-gcm, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, fast-open=false, udp-relay=false, tag=Sample-J

vmess=vmess-b.example.com:80, method=none, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, fast-open=false, udp-relay=false, tag=Sample-K
```


- over-tls 类型

```
vmess=vmess-over-tls.example.com:443, method=none, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs-host=vmess-over-tls.example.com, obfs=over-tls, tls-verification=true, fast-open=false, udp-relay=false, tag=Sample-L
```


##### http(s) 类型

```
http=http.example.com:80, username=name, password=pwd, fast-open=false, udp-relay=false, tag=http

http=https.example.com:443, username=name, password=pwd, over-tls=true, tls-verification=true, tls-host=example.com, tls-verification=true, fast-open=false, udp-relay=false, tag=http-tls
```

##### socks5 类型节点

```
socks5=example.com:80,fast-open=false, udp-relay=false, tag=socks5-01

socks5=example.com:80, username=name, password=pwd, fast-open=false, udp-relay=false, tag=socks5-02

socks5=example.com:443, username=name, password=pwd, over-tls=true, tls-host=example.com, tls-verification=true, fast-open=false, udp-relay=false, tag=socks5-tls-01

socks5=example.com:443, username=name, password=pwd, over-tls=true, tls-host=example.com, tls-verification=true, tls-pubkey-sha256=eb5ec6684564fd0d04975903ed75342d1b9fdc2096ea54b4cf8caf4740f4ae25, fast-open=false, udp-relay=false, tag=socks5-tls-02
```

##### trojan 类型

支持 over-tls 以及 websockets，支持 UDP

```
trojan=example.com:443, password=pwd, over-tls=true, tls-verification=true, fast-open=false, udp-relay=true, tag=trojan-tls-01

trojan=example1.com:443, password=pwd, over-tls=true, tls-host=example.com, tls-verification=true, fast-open=false, udp-relay=false, tag=trojan-tls-02

trojan=192.168.1.1:443, password=pwd, obfs=wss, obfs-host=example.com, obfs-uri=/path, udp-relay=true, tag=trojan-wss-05
```

##### vless 类型

<!-- prettier-ignore -->
!!! 注意
    需 Version ≥ 1.5.0

```
vless=example.com:80, method=none, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, fast-open=false, udp-relay=false, tag=vless-01

vless=example.com:443, method=none, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs=over-tls, fast-open=false, udp-relay=false, tag=vless-tls-01

vless=example.com:80, method=none, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs=http, obfs-host=bing.com, obfs-uri=/resource/file, fast-open=false, udp-relay=false, server_check_url=http://www.apple.com/generate_204, tag=vless-http

vless=192.168.1.1:443, method=none, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs=over-tls, obfs-host=example.com, fast-open=false, udp-relay=false, tag=vless-tls-02

vless=192.168.1.1:443, method=none, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs=over-tls, obfs-host=example.com, tls13=true, fast-open=false, udp-relay=false, tag=vless-tls-03

vless=example.com:80, method=none, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs=ws, obfs-uri=/ws, fast-open=false, udp-relay=false, tag=vless-ws-01

vless=192.168.1.1:80, method=none, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs=ws, obfs-host=example.com, obfs-uri=/ws, fast-open=false, udp-relay=false, tag=vless-ws-02

vless=example.com:443, method=none, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs=wss, obfs-uri=/ws, fast-open=false, udp-relay=false, tag=vless-ws-tls-01

vless=192.168.1.1:443, method=none, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs=wss, obfs-host=example.com, obfs-uri=/ws, fast-open=false, udp-relay=false, tag=vless-ws-tls-02

vless=192.168.1.1:443, method=none, password=23ad6b10-8d1a-40f7-8ad0-e3e35cd32291, obfs=wss, obfs-host=example.com, obfs-uri=/ws, tls13=true, fast-open=false, udp-relay=false, tag=vless-ws-tls-03
```


### 2.3 节点延迟

该设置一般通过配置文件`[general]`下的参数进行配置

```
[general]
network_check_url = http://wifi.vivo.com.cn/generate_204
server_check_url = http://1.1.1.1/generate_204
; server_check_user_agent = Agent/1.0
server_check_timeout = 5000
```

- `network_check_url` 网络检查 URL 设置；
- `server_check_url` 代理服务器网络检查 URL 设置；
- `server_check_user_agent` 代理服务器检查的 User Agent 自定义设置；
- `server_check_timeout` 测试超时设置，单位为 ms；

#### 2.3.1 触发节点延迟测试

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/Server_tets.webp" width="600">

- 单击节点资源图标/策略组图标，即可对该项目下的所有节点进行测试
- 节点资源(PROXY)区域,长按节点，即可对单个节点进行测试

Quantumult 使用 HEAD 方法向 `server_check_url` 发送 HTTP 请求来测试代理的状态，结果应该是两个延迟：

- 第一个是到代理服务器的 TCP 握手
- 第二个是 HTTP 延迟（Quantumult 成功接收到 HTTP 之间的持续时间）来自 server_check_url 的响应和 Quantumult 开始将 HTTP 请求发送到 server_check_url）

<!-- prettier-ignore -->
!!! 注意
    如果 `[server_local]` 或 `[server_remote]` 区块中的节点有自己的`server_check_url`，则将使用其自己的`server_check_url`，而不是`[general]`区块中的`server_check_url`。


### 2.4 节点标识说明

节点后标识：

- 🅰：Vmess 协议的AEAD（aead=true）
- 🅵：tcp-fast-open 开启（fast-open=true）
- 🆄：UDP 开启（udp-relay=true）

- 🆂：tls，开启验证（tls-verification=true）
- 🅂：tls，未开启验证（tls-verification=false）
- 🅿：tls-cert-sha256 / tls-pubkey-sha256 自定义验证参数

- 🆄 🅵 显示仅代表配置开启，不代表服务端支持
- 黑色⚡️表示 TFO 触发成功

节点前标识：

- ⚠︎：标识有节点名称重复



### 2.5 节点资源图标

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/ICON.webp" width="600">

<!-- prettier-ignore -->
!!! 提示
    QX 节点资源区域的图标只支持单色，策略组区域支持彩色；
    
    图标分辨率限制144x144、108x108.

- 长按策略组/节点资源 → 图标，即可通过 UI 修改图标；
- 修改配置文件的 `img-url` 即可自定义图标
- 点此查看[图标集](../quantumultx/icon.md)

