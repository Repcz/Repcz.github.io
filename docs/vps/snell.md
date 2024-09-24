# 快速部署 SNELL 协议

## 脚本安装


### SNELL 一键脚本

```bash
wget -O snell.sh --no-check-certificate https://git.io/Snell.sh && chmod +x snell.sh && ./snell.sh
```



## Docker 部署


### 创建文件夹

```bash
mkdir -p /root/snelldocker/snell-conf
```

### 写入 Docker Compose 文件

```bash
cat > /root/snelldocker/docker-compose.yml << EOF
services:
  snell:
    image: accors/snell:latest
    container_name: snell
    restart: always
    network_mode: host
    volumes:
      - ./snell-conf/snell.conf:/etc/snell-server.conf
    environment:
      - SNELL_URL=https://dl.nssurge.com/snell/snell-server-v4.1.1-linux-amd64.zip
EOF
```

### 写入 SNELL 配置文件

```bash
cat > /root/snelldocker/snell-conf/snell.conf << EOF
[snell-server]
listen = 0.0.0.0:65110
psk = jy7jbw6yFWikg2uS
tfo = true
version = 4
dns = 1.1.1.1, 8.8.8.8
EOF
```




### 启动/删除 SNELL Docker


1.启动 `Docker compose`

```bash
cd /root/snelldocker
```

```bash
docker compose up -d
```

2.停止删除 `Docker compose`

```bash
docker compose down
```
