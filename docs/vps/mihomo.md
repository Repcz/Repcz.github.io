# mihomo Docker 部署

## 安装 Docker

```bash
curl -sS -O https://raw.githubusercontent.com/kejilion/sh/main/kejilion.sh && chmod +x kejilion.sh && ./kejilion.sh
```

## 部署 mihomo Docker compose

1. 新建目录

```bash
mkdir -p /root/mihomo
```

2. 写入 `Docker compose` 文件

```bash
cat > /root/mihomo/docker-compose.yml << EOF
services:
  mihomo:
    container_name: mihomo
    image: metacubex/mihomo:latest
    restart: always
    pid: host
    ipc: host
    network_mode: host
    cap_add:
      - ALL
    security_opt:
      - apparmor=unconfined
    volumes:
      - ~/mihomo:/root/.config/mihomo
      - /dev/net/tun:/dev/net/tun
      # 共享host的时间环境
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro

EOF
```

## 写入 mihomo 配置

### 服务端配置文件

此配置文件同时搭建 `SS2022+anyTLS` 节点，请根据需要自行修改。

```bash
cat > /root/mihomo/config.yaml << EOF

mixed-port: 65222 # HTTP(S) 和 SOCKS 代理混合端口
tcp-concurrent: true # TCP 并发连接所有 IP, 将使用最快握手的 TCP
allow-lan: false # 允许局域网连接
ipv6: false # 开启 IPv6 总开关，关闭阻断所有 IPv6 链接和屏蔽 DNS 请求 AAAA 记录
log-level: info # 日志等级 silent/error/warning/info/debug

hosts:
  "localhost": 127.0.0.1

dns:
  enable: true
  listen: :65223 # 开启 DNS 服务器监听
  ipv6: false # false 将返回 AAAA 的空结果
  use-hosts: true # 查询 hosts
  enhanced-mode: redir-host
  default-nameserver: [1.1.1.1]
  nameserver: [8.8.4.4, 1.1.1.1]
  
rules:
  - MATCH,DIRECT

listeners: #搭建代理节点

  - name: SS2022
    type: shadowsocks
    port: 60111
    listen: 0.0.0.0
    cipher: 2022-blake3-aes-128-gcm
    password: +53N4IrJVaULrc7h4xGN6g==
    udp: true
    shadow-tls:
       enable: true 
       version: 3
       users:
         - name: 1
           password: +53N4IrJVaULrc7h4xGN6g==
       handshake:
         dest: www.icloud.com:443

  - name: anytls
    type: anytls
    port: 60112
    listen: 0.0.0.0
    users:
      username1: +53N4IrJVaULrc7h4xGN6g==
    certificate: ./server.crt # 证书 PEM 格式，或者 证书的路径
    private-key: ./server.key

EOF
```

#### `SS2022` 密码生成

`SS2022` 对密码有格式及长度要求：

| 加密格式 | 密码长度 
| ----------- | ----------- |
| `2022-blake3-aes-128-gcm`	| 16 |
| `2022-blake3-aes-256-gcm`	| 32 |
| `2022-blake3-chacha20-poly1305` |	32 |

生成密码可参考如下命令：

```bash
openssl rand --base64 32
openssl rand --base64 16
```

#### `hysteria2` 或 `anyTLS` 节点需自签证书

```bash
openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) -keyout /root/mihomo/server.key -out /root/mihomo/server.crt -subj "/CN=bing.com" -days 36500 && sudo chown mihomo /root/mihomo/server.key && sudo chown mihomo /root/mihomo/server.crt
```

## 启动/删除 mihomo Docker

1. 启动容器

```bash
cd /root/mihomo && docker compose up -d
```

2. 停止删除容器

```bash
cd /root/mihomo && docker compose down
```
