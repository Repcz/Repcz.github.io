# 快速部署 Hysteria2 协议

## 官方脚本安装


### 安装/升级

```bash
bash <(curl -fsSL https://get.hy2.sh/)
```

### 卸载

```bash
bash <(curl -fsSL https://get.hy2.sh/) --remove
```

### 写入 Bing 自签证书

```bash
openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) -keyout /etc/hysteria/server.key -out /etc/hysteria/server.crt -subj "/CN=bing.com" -days 36500 && sudo chown hysteria /etc/hysteria/server.key && sudo chown hysteria /etc/hysteria/server.crt
```

### 写入 Hysteria 配置文件

```bash
cat > /etc/hysteria/config.yaml << EOF
listen: :443 #监听端口，可以自定义

#使用自签证书
tls:
  cert: /etc/hysteria/server.crt
  key: /etc/hysteria/server.key

auth:
  type: password
  password: 123456 #设置认证密码
  
masquerade:
  type: proxy
  proxy:
    url: https://bing.com #伪装网址
    rewriteHost: true

quic:
  initStreamReceiveWindow: 26843545
  maxStreamReceiveWindow: 26843545
  initConnReceiveWindow: 67108864
  maxConnReceiveWindow: 67108864
  maxIdleTimeout: 30s
  maxIncomingStreams: 1024
  disablePathMTUDiscovery: false

bandwidth:
  up: 100 mbps       # 按需修改
  down: 30 mbps       # 按需修改

ignoreClientBandwidth: false

udpIdleTimeout: 60s

EOF
```

### 重启 Hysteria

```bash
systemctl restart hysteria-server.service
```

### 设置开机自启

```bash
systemctl enable hysteria-server.service
```

---

??? note "启动、重启、查看状态、停止 Hy2 指令"
    启动Hysteria2

    ```bash
    systemctl start hysteria-server.service
    ```

    重启Hysteria2

    ```bash
    systemctl restart hysteria-server.service
    ```

    查看Hysteria2状态

    ```bash
    systemctl status hysteria-server.service
    ```

    停止Hysteria2

    ```bash
    systemctl stop hysteria-server.service
    ```

    设置开机自启

    ```bash
    systemctl enable hysteria-server.service
    ```

    查看日志
    
    ```bash
    journalctl -u hysteria-server.service
    ```

## Docker 部署


### 创建文件夹

```bash
mkdir /root/hysteria2
```

```bash
cd /root/hysteria2
```

### 写入 Docker Compose 文件

```bash
cat > /root/hysteria2/docker-compose.yml << EOF
version: "3.9" 
services:
  hysteria:
    image: tobyxdd/hysteria
    container_name: hy2
    restart: always
    network_mode: host
    volumes:
      - .:/etc/hysteria
    command: ["server", "-c", "/etc/hysteria/hysteria.yaml"]
EOF
```

### 写入 Hysteria 配置文件

```bash
cat > /root/hysteria2/hysteria.yaml << EOF
listen: :443 #监听端口，可以自定义

#使用自签证书
tls:
  cert: /etc/hysteria/server.crt
  key: /etc/hysteria/server.key

auth:
  type: password
  password: 123456 #设置认证密码
  
masquerade:
  type: proxy
  proxy:
    url: https://bing.com #伪装网址
    rewriteHost: true

quic:
  initStreamReceiveWindow: 26843545
  maxStreamReceiveWindow: 26843545
  initConnReceiveWindow: 67108864
  maxConnReceiveWindow: 67108864
  maxIdleTimeout: 30s
  maxIncomingStreams: 1024
  disablePathMTUDiscovery: false

bandwidth:
  up: 100 mbps       # 按需修改
  down: 30 mbps       # 按需修改

ignoreClientBandwidth: false

udpIdleTimeout: 60s

EOF
```


### 写入 Bing 自签证书

```bash
openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) -keyout /root/hysteria2/server.key -out /root/hysteria2/server.crt -subj "/CN=bing.com" -days 36500 && sudo chown hysteria /root/hysteria2/server.key && sudo chown hysteria /root/hysteria2/server.crt
```

### 启动/删除 Hysteria Docker


1.启动 `Docker compose`

```bash
docker compose up -d
```

2.停止删除 `Docker compose`
```bash
docker compose down
```

## 设置端口跳跃


[官方文档](https://v2.hysteria.network/zh/docs/advanced/Port-Hopping/)

来源：https://blog.passall.us/archives/1199

<!-- prettier-ignore -->
!!! 注意
    `20000:40000` 为跳跃端口，`9443` 为 HY2 监听的端口

```bash
apt install iptables-persistent
```

```bash
iptables -t nat -A PREROUTING -i eth0 -p udp --dport 20000:40000 -j DNAT --to-destination :9443
```

```bash
ip6tables -t nat -A PREROUTING -i eth0 -p udp --dport 20000:40000 -j DNAT --to-destination :9443
```

```bash
netfilter-persistent save
```