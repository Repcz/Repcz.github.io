# Mihomo Docker 部署

## 安装 Docker

```
curl -sS -O https://raw.githubusercontent.com/kejilion/sh/main/kejilion.sh && chmod +x kejilion.sh && ./kejilion.sh
```

脚本 → 6 `Docker 管理` → 1 `安装 Docker`

如果是国内服务器，可能需要更换 `Docker源`：脚本 → 6 `Docker 管理` → 8 `更换 Docker 源`

## 部署 Mihomo Docker compose

1.新建目录

```bash
mkdir -p /root/mihomo
```

2.写入 `Docker compose` 文件

```bash
cat > /root/mihomo/docker-compose.yml << EOF
services:
  clash:
    container_name: clash-meta
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

```bash
cat > /root/mihomo/config.yml << EOF

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
    'geosite:cn': system
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

## 启动/删除 Mihomo Docker

1.进入目录

```bash
cd /root/mihomo
```

2.启动 `Docker compose`

```bash
docker compose up -d
```

3.或者 停止删除 `Docker compose`

```bash
docker compose down
```
