# 部署 Caddy 反向代理工具

[Caddy 官方中文文档](https://caddy2.dengxiaolong.com/docs/)

## 官方脚本部署

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
```

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
```

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
```

```bash
sudo apt update
```

```bash
sudo apt install caddy
```

??? note "Caddy 启动、停止、重启、查看状态"
    启动

    ```bash
    sudo systemctl start caddy
    ```

    停止

    ```bash
    sudo systemctl stop caddy
    ```

    重启

    ```bash
    sudo systemctl reload caddy
    ```

    查看状态
    
    ```bash
    systemctl status caddy
    ```


