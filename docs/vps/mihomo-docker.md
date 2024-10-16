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
