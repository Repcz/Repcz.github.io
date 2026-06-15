# 使用 Caddy 管理证书 + Caddy 转发 SNI 至 sing-box

## Caddy 安装 l4 转发

```bash
caddy add-package github.com/mholt/caddy-l4
sudo systemctl restart caddy
```

## Caddyfile 配置

```cafddyfile
{
    servers :443 {
        listener_wrappers {
            layer4 {
                @anytls tls sni your.domain
                route @anytls {
                    proxy 127.0.0.1:8443
                }
            }
            tls
        }
    }
}

your.domain {
    respond 204
}
```

## sing-box anyTLS 入站

```json
"inbounds": [
        {
            "type": "anytls",
            "tag": "anytls-in",
            "listen": "127.0.0.1",
            "listen_port": 8443,
            "users": [
                {
                    "name": "singbox",
                    "password": "your_passwd"
                }
            ],
            "padding_scheme": [],
            "tls": {
                "enabled": true,
                "server_name": "your.domain",
                "certificate_path": "/var/lib/caddy/.local/share/caddy/certificates/acme-v02.api.letsencrypt.org-directory/your.domain/your.domain.crt",
                "key_path": "/var/lib/caddy/.local/share/caddy/certificates/acme-v02.api.letsencrypt.org-directory/your.domain/your.domain.key"
            }
        }
    }
```

对应的 Surge 节点格式：

```
Node_name = anytls, your_server_address, 443, password=your_passwd, sni=your.domain
```
