# Uptime kuma

- 创建文件夹

```bash
mkdir -p /root/uptime
```

- 写入配置

```bash
cat > /root/uptime/docker-compose.yml << EOF
services:
  uptime-kuma:
    image: louislam/uptime-kuma:beta
    container_name: uptime-kuma
    restart: always
    network_mode: host
    environment:
      HOST: "127.0.0.1"
      PORT: 3001
    volumes:
      - ./data:/app/data
      - /run/docker.sock:/run/docker.sock
EOF
```

- 启动容器

```bash
cd /root/mihomo && docker compose up -d
```

- 停止删除容器

```bash
cd /root/mihomo && docker compose down
```
