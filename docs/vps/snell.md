# 快速部署 SNELL 协议

## 脚本安装


### SNELL 一键脚本

```bash
wget -O snell.sh --no-check-certificate https://git.io/Snell.sh && chmod +x snell.sh && ./snell.sh
```

## Docker 部署


### 创建文件夹

```bash
mkdir snell && cd snell
```

### 写入 Docker Compose 文件

```bash
cat > docker-compose.yaml << EOF
services:
  snell:
    image: vocrx/snell-server:latest
    container_name: snell
    restart: always
    network_mode: host
    environment:
      - PORT=65110
      - PSK=jy7jbw6yFWikg2uS
      - DNS=1.1.1.1,8.8.8.8
EOF
```

### 自定义 SNELL 配置文件

修改 `environment` 下的参数, 即可自定义 SNELL 配置

```yaml
services:
  snell:
    image: vocrx/snell-server:latest
    container_name: snell
    restart: always
    network_mode: host
    environment:
      - PORT=自定义使用的端口, 仅host模式下生效, 不写则随机。
      - PSK=节点密码, 不写则随机。
      - IPV6=true/false, 不写默认为false。
      - DNS=8.8.8.8,1.1.1.1, 不写为系统默认
      - VERSION=v4.1.1, 自定义二进制文件版本, 不写则默认最新版
      - OBFS=http,默认为空,写此条必须配置HOST
      - HOST=icloud.com,默认为空
```


### 启动/删除 SNELL Docker


1.启动 `Docker compose`

```bash
docker compose up -d && cd
```

2.停止删除 `Docker compose`

```bash
docker compose down && cd
```
