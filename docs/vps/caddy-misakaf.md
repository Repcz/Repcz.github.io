# Caddy Docker 反向代理 MisakaF Emby

## 首要条件

- 准备一个域名
- 将域名 DNS记录 指向服务器 IP


## 搭建Caddy Docker

拉取官方 Caddy 镜像

```bash
docker pull caddy:latest
```

## 写入 Caddyfile

创建 caddyflie 目录
```bash
mkdir -p /etc/caddy
```

写入 caddyfile 文件，注意修改 `misakaf.your.domain` 为你的域名，`emby.misakaf.org` 修改为你的emby账号对应的服务器

!!! info ""
    此 caddyfile 配置文件参考 [MisakaF-nginx-reProxy](https://github.com/lateautumn2/MisakaF-nginx-reProxy)，并通过 GPT 修改而来


```bash
cat > /etc/caddy/Caddyfile << EOF

EOF
```

## 启动容器

- 创建一个挂载目录

```bash
mkdir -p /srv /etc/caddy
```

- 启动 Caddy 容器

```bash
docker run -d \
    --name caddy \
    -p 80:80 \
    -p 443:443 \
    -v /etc/caddy/Caddyfile:/etc/caddy/Caddyfile \
    -v caddy_data:/data \
    -v caddy_config:/config \
    caddy:latest
```

查看 Docker 日志
```bash
docker logs caddy
```