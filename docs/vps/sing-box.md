# sing-box 服务端

## 部署

### :material-console: Docker 安装

```bash
docker run -d \
  -v /etc/sing-box:/etc/sing-box/ \
  --name=sing-box \
  --restart=always \
  ghcr.io/sagernet/sing-box \
  -D /var/lib/sing-box \
  -C /etc/sing-box/ run
```

### :material-download-box: 手动安装

=== ":material-debian: Debian"

    ```bash
    curl -fsSL https://sing-box.app/install.sh | sh
    ```

#### :material-book-multiple: 服务管理

对于带有 [systemd][systemd] 的 Linux 系统，通常安装已经包含 sing-box 服务，
您可以使用以下命令管理服务：

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

[systemd]: https://systemd.io/


## 服务端配置

配置文件位置: `/etc/sing-box/config.json`

### SS2022+ShadowTLS+Padding 协议

配置如下，来源：https://t.me/ztvps/95

```json
{
    "log": {
        "disabled": false,
        "level": "info",
        "timestamp": true
    },
    "inbounds": [
        {
            "type": "shadowtls",
            "tag": "Shadowsocks+ShadowTLS+Padding", // 可改，给节点一个备注
            "listen": "::", // 可改，未限制，监听所有来访IP，来访IP只要有正确的配置，即可连接
            "listen_port": 123, // 改，节点的端口
            "version": 3, // 可改，ShadowTLS 协议版本，默认为1，建议启用3
            "users": [
                {
                    "name": "singbox", // 可改，多用户配置时用作区分
                    "password": "密码" // 改，使用 sing-box generate rand --base64 16 生成（无法使用请补全sing-box所在的绝对路径 /绝对/路径/sing-box generate rand --base64 16）
                }
            ],
            "handshake": {
                "server": "域名", // 改，填想要偷取证书的域名
                "server_port": 443 // 可改，偷取证书的域名的端口，绝大部分情况为443
            },
            "strict_mode": true, // 可改，ShadowTLS 严格模式，仅在ShadowTLS版本为3时可用
            "detour": "shadowsocks-shadowtls-in" // 可改，转发到指定ss节点，tag要和下面一致，不然会错误
        },
        {
            "type": "shadowsocks",
            "tag": "shadowsocks-shadowtls-in", // 可改，转发到到的ss节点，tag要和上面一致，不然会错误
            "listen": "127.0.0.1", // 可改，除非你也想使用ShadowTLS+Shadowsocks同时，直接连接Shadowsocks，那么就可以把listen内容直接改成 :: 即可
            "sniff": true, // 开启嗅探
            "sniff_override_destination": false, // 无特殊需求，此项可不开，具体请看Sing-Box官方文档
            "method": "2022-blake3-aes-128-gcm", // 可改
            "password": "密码", // 改，使用 sing-box generate rand --base64 16 生成（无法使用请补全sing-box所在的绝对路径 /绝对/路径/sing-box generate rand --base64 16）
            "multiplex": {
                "enabled": true,
                "padding": true // 入站开启Padding功能，使服务器仅接收padding后的数据，非padding数据拒绝入站
            }
        }
    ],
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



#### `SS2022` 密码生成

`SS2022` 对密码有格式及长度要求：

| 加密格式 | 密码长度 
| ----------- | ----------- |
| `2022-blake3-aes-128-gcm`	| 16 |
| `2022-blake3-aes-256-gcm`	| 32 |
| `2022-blake3-chacha20-poly1305` |	32 |

生成密码可参考如下命令：

```bash
openssl rand --base64 32
openssl rand --base64 16
```

或

```bash
sing-box generate rand --base64 16
sing-box generate rand --base64 32
```