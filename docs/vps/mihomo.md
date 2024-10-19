# Mihomo Docker 部署

## 安装 Docker

```bash
curl -sS -O https://raw.githubusercontent.com/kejilion/sh/main/kejilion.sh && chmod +x kejilion.sh && ./kejilion.sh
```

脚本 → 6 `Docker 管理` → 1 `安装 Docker`

如果是国内服务器，可能需要更换 `Docker源`：脚本 → 6 `Docker 管理` → 8 `更换 Docker 源`

## 部署 Mihomo Docker compose

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

## 写入 Mihomo 配置

请区分作为 `客户端` 和 `服务端` 的配置文件。

### 客户端配置文件

```bash
cat > /root/mihomo/config.yaml << EOF

proxy-providers:
  Subscribe: {url: http://your-service-provider, path: ./proxy-providers/Sub.yaml, type: http, interval: 86400, health-check: {enable: true, url: http://connectivitycheck.gstatic.com/generate_204, interval: 1800}}

mixed-port: 7893
tcp-concurrent: true
allow-lan: true
ipv6: false
log-level: info
unified-delay: true
global-client-fingerprint: chrome
find-process-mode: strict
external-controller: 127.0.0.1:56110
secret: "your-external-controller-API-secret"
external-ui: ui
external-ui-url: "https://github.com/MetaCubeX/metacubexd/archive/refs/heads/gh-pages.zip"

geodata-mode: true
geox-url:
  geoip: "https://mirror.ghproxy.com/https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geoip-lite.dat"
  geosite: "https://mirror.ghproxy.com/https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geosite.dat"
  mmdb: "https://mirror.ghproxy.com/https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/country-lite.mmdb"
  asn: "https://mirror.ghproxy.com/https://github.com/xishang0128/geoip/releases/download/latest/GeoLite2-ASN.mmdb"

profile:
  store-selected: true 
  store-fake-ip: true  

sniffer:
  enable: true
  sniff:
    HTTP:
      ports: [80, 8080-8880]
      override-destination: true
    TLS:
      ports: [443, 8443]
    QUIC:
      ports: [443, 8443]

tun:
  enable: true
  stack: mixed
  dns-hijack: [any:53]
      
dns:
  enable: true
  ipv6: false
  enhanced-mode: fake-ip
  listen: :1053
  fake-ip-range: 198.18.0.1/16
  fake-ip-filter: ['+.lan', '*', '+.local']
  default-nameserver: [223.5.5.5, 119.29.29.29, system]
  nameserver: [223.5.5.5, 119.29.29.29]
  nameserver-policy:
    'geosite:cn': [system]
    'geosite:gfw,geolocation-!cn': [quic://223.5.5.5, quic://223.6.6.6, https://1.12.12.12/dns-query, https://120.53.53.53/dns-query]

proxy-groups:
  - {name: Proxy, type: fallback, include-all: true}

rules:
  - PROCESS-PATH,/opt/nezha/agent/nezha-agent,DIRECT
  - GEOSITE,openai,Proxy
  - GEOSITE,category-games,Proxy
  - GEOSITE,github,Proxy
  - GEOSITE,telegram,Proxy
  - GEOSITE,twitter,Proxy
  - GEOSITE,microsoft,Proxy
  - GEOSITE,youtube,Proxy
  - GEOSITE,google,Proxy
  - GEOSITE,private,DIRECT
  - GEOSITE,cn,DIRECT
  - GEOSITE,geolocation-!cn,Proxy
  - GEOIP,telegram,Proxy
  - GEOIP,twitter,Proxy
  - GEOIP,google,Proxy
  - GEOIP,private,DIRECT
  - GEOIP,lan,DIRECT
  - GEOIP,CN,DIRECT
  - MATCH,Proxy

EOF
```

### 服务端配置文件

此配置文件同时搭建 `SS 2022` 和 `hysteria2` 节点，请根据需要自行修改。

```bash
cat > /root/mihomo/config.yaml << EOF

mixed-port: 65222 # HTTP(S) 和 SOCKS 代理混合端口
tcp-concurrent: true # TCP 并发连接所有 IP, 将使用最快握手的 TCP
allow-lan: false # 允许局域网连接
ipv6: true # 开启 IPv6 总开关，关闭阻断所有 IPv6 链接和屏蔽 DNS 请求 AAAA 记录
log-level: info # 日志等级 silent/error/warning/info/debug

hosts:
  "localhost": 127.0.0.1

dns:
  enable: true
  listen: :65223 # 开启 DNS 服务器监听
  ipv6: true # false 将返回 AAAA 的空结果
  use-hosts: true # 查询 hosts
  enhanced-mode: redir-host
  nameserver: [8.8.4.4, 9.9.9.12, 208.67.220.220, 94.140.14.141]

rules:
  - MATCH,DIRECT

listeners: #搭建代理节点

  - name: SS2022
    type: shadowsocks
    port: 65112
    listen: "::"
    cipher: 2022-blake3-aes-256-gcm
    password: SaAj4IC+cHEyWoCaUXeNBE+A8DcqKRsOELe4FOuuNJE=
    udp: true
    udp-over-tcp: false
    ip-version: ipv4-prefer

    smux:
      enabled: true
      protocol: h2mux
      max-connections: 0
      min-streams: 0
      max-streams: 1
      statistic: true
      only-tcp: false
      padding: true

  - name: hy2
    type: hysteria2
    port: 65111
    listen: "::"
    users:
      user1: password1
      user2: password2
    up: 200
    down: 30
    masquerade: "https://bing.com"
    certificate: ./server.crt
    private-key: ./server.key

EOF
```

#### `SS2022` 密码生成

- `2022-blake3-aes-128-gcm`	16位密码长度
- `2022-blake3-aes-256-gcm`	32位密码长度
- `2022-blake3-chacha20-poly1305`	32位密码长度

```bash
openssl rand --base64 <密码长度>
```

#### `hysteria2` 节点需自签证书

```bash
openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) -keyout /root/mihomo/server.key -out /root/mihomo/server.crt -subj "/CN=bing.com" -days 36500 && sudo chown mihomo /root/mihomo/server.key && sudo chown mihomo /root/mihomo/server.crt
```

## 启动/删除 Mihomo Docker

1. 进入目录

```bash
cd /root/mihomo
```

2. 启动容器

```bash
docker compose up -d
```

3. 停止删除容器

```bash
docker compose down
```
