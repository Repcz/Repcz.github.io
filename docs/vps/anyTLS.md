# 安装 sing-box

## 官方脚本安装

安装 beta 版本：

```bash
curl -fsSL https://sing-box.app/install.sh | sh -s -- --beta
```

安装完成后，脚本会自动：

- 下载对应系统架构的 sing-box 二进制文件
- 安装为 `systemd` 服务
- 创建配置文件目录 `/etc/sing-box/`


## 写入配置文件

本仓库提供了现成的 [`anyTLS` 服务端配置](https://github.com/Repcz/Tool/blob/X/sing-box/v1.14.x/Server/anyTLS/config.json)：

请下载配置后修改并保持上传至`/etc/sing-box/`，以下内容仅做示例，不定期更新

```json
{
    "log": {
        "disabled": false,
        "level": "info",
        "timestamp": true
    },

    "certificate_providers": [
        {
            "type": "acme",
            "tag": "ip_cert",
            "domain": [
                "your_server_ip"
            ],
            "email": "admin@example.com"
        }
    ],

    "dns": {
        "servers": [
            {
                "tag": "cloudflare",
                "type": "udp",
                "server": "1.1.1.1"
            }
        ],
        "rules": [
            {
                "query_type": [
                    "A",
                    "AAAA"
                ],
                "action": "route",
                "server": "cloudflare"
            }
        ],
        "strategy": "prefer_ipv4",
        "final": "cloudflare"
    },

    "inbounds": [
        {
            "type": "anytls",
            "tag": "anytls-in",
            "listen": "::",
            "listen_port": 443,
            "users": [
                {
                    "name": "singbox",
                    "password": ""
                }
            ],
            "padding_scheme": [],
            "tls": {
                "enabled": true,
                "server_name": "your_server_ip",
                "certificate_provider": "ip_cert"
            }
        }
    ],

    "route": {
        "rules": [
            {
                "inbound": [
                    "anytls-in"
                ],
                "action": "sniff"
            }
        ],
        "default_domain_resolver": "cloudflare"
    },

    "outbounds": [
        {
            "type": "direct"
        }
    ],

    "ntp": {
        "enabled": true,
        "server": "time.apple.com",
        "server_port": 123,
        "interval": "30m"
    }
}
```

### 必须修改的字段

| 字段 | 位置 | 说明 |
|------|------|------|
| `your_server_ip` | `certificate_providers[0].domain[0]` | 替换为你的 **VPS 公网 IP** |
| `your_server_ip` | `inbounds[0].tls.server_name` | 替换为你的 **VPS 公网 IP** |
| `admin@example.com` | `certificate_providers[0].email` | 替换为你的 **邮箱**（Let's Encrypt 注册用） |
| `""`（password） | `inbounds[0].users[0].password` | 设置一个 **连接密码**，客户端连接时需要 |

---

## 启动 sing-box

### 检查配置语法

```bash
sing-box check -c /etc/sing-box/config.json
```

如果输出为空且无报错，说明配置正确。

### 服务管理

安装完成后执行 `sudo systemctl enable sing-box && sudo systemctl restart sing-box` 启动 sing-box

| 行动   | 命令                                            |
|------|-----------------------------------------------|
| 启用   | `sudo systemctl enable sing-box`              |
| 禁用   | `sudo systemctl disable sing-box`             |
| 启动   | `sudo systemctl start sing-box`               |
| 停止   | `sudo systemctl stop sing-box`                |
| 强行停止 | `sudo systemctl kill sing-box`                |
| 重新启动 | `sudo systemctl restart sing-box`             |
| 查看日志 | `sudo journalctl -u sing-box --output cat -e` |
| 实时日志 | `sudo journalctl -u sing-box --output cat -f` |