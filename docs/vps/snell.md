## 关于 Snell v6

Snell v6 是 Surge 团队于 2026 年 6 月发布的下一代 Snell 协议。

其核心变化在于：**从单一的协议层流量特征，转向 PSK 派生的部署级多样性**。

具体来说，Snell v6 基于一个包含 42 个特征参数和 13 类填充/流量整形策略的「协议画像」(protocol profile)。**不同的 PSK 会派生出不同的协议画像**，从而使得不同部署呈现不同的流量特征。这意味着：

- 不再存在所有 Snell 部署共享的固定流量指纹
- 攻击者无法通过单一特征匹配来识别 Snell
- 用户无需手动配置任何底层参数，仅需确保 PSK 不同即可

> 官方博客：[Introducing Snell v6: Deployment-Level Protocol Diversity](https://nssurge.com/blog/snell-v6/)

---

## 一键脚本（第三方）

[第三方作者](https://github.com/xOS/Snell)维护的 Snell 一键管理脚本，部署原生 Snell 独立服务：

```bash
wget -O snell.sh --no-check-certificate https://git.io/Snell.sh && chmod +x snell.sh && ./snell.sh
```

执行后可按菜单操作，支持安装、卸载、修改配置等。

---

## 部署 Snell（sing-box）

### 为什么 sing-box 支持 Snell

> 原文：[sing-snell](https://github.com/SagerNet/sing-snell#why)

Surge 认为闭源和不扩散可以保持协议的隐蔽性，但在 2026 年这已不再可能。考虑到 Snell 仍然拥有其他随机流量协议不具备的优势——例如支持多路复用并保留完整 TCP 语义，以及流量特征多样性——因此我们实现了它，而不是重复造轮子。

### 安装 sing-box

安装 beta 版本：

```bash
curl -fsSL https://sing-box.app/install.sh | sh -s -- --beta
```

安装完成后，脚本会自动：

- 下载对应系统架构的 sing-box 二进制文件
- 安装为 `systemd` 服务
- 创建配置文件目录 `/etc/sing-box/`

### 写入配置文件

本仓库提供了现成的 [Snell v6 服务端配置](https://github.com/Repcz/Tool/raw/X/sing-box/v1.14.x/Server/Snell/config.json)：

请下载配置后修改并保持上传至 `/etc/sing-box/`，以下内容仅做示例，不定期更新：

```json
{
    "log": {
        "disabled": false,
        "level": "info",
        "timestamp": true
    },

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
            "type": "snell",
            "tag": "snell-in",
            "listen": "::",
            "listen_port": 443,
            "version": 6,
            "psk": "",
            "mode": "default"
        }
    ],

    "route": {
        "rules": [
            {
                "inbound": [
                    "snell-in"
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

#### 必须修改的字段

| 字段 | 位置 | 说明 |
|------|------|------|
| `psk` | `inbounds[0].psk` | 设置一个 **预共享密钥**，长度需 12~255 字节。建议使用 `sing-box generate rand --base64 $((RANDOM % 244 + 12))` 生成 |

#### 可选字段说明

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `version` | `5` / `6` | — | **必填**。Snell 协议版本。v5 支持 HTTP 混淆（`obfs_mode`），v6 以流量整形（`mode`）取而代之 |
| `mode` | string | `"default"` | 仅 v6。流量整形模式：`"default"` / `"unshaped"` / `"unsafe-raw"`。推荐保持 `"default"` |
| `obfs_mode` | string | `"none"` | 仅 v5。HTTP 混淆模式：`"none"` / `"http"` |
| `users` | array | — | 多用户模式。每项包含 `name`（可选，用于日志）和 `userkey`（用户密钥）。设置后顶层 `psk` 作为服务器密钥 |

#### `mode` 说明（v6）

| 模式 | 说明 |
|------|------|
| `"default"` | 标准流量整形模式。PSK 派生协议画像，产生部署级流量特征多样性。**推荐使用** |
| `"unshaped"` | 原始 Snell v4 兼容模式。不应用流量整形，回退到随机数据流特征 |
| `"unsafe-raw"` | 明文调试模式。**仅用于调试，不应在生产环境使用** |

> 注意：`unsafe-raw` 模式下不会对流量进行加密，仅应在本地测试时使用。

#### 多用户模式示例

```json
{
    "inbounds": [
        {
            "type": "snell",
            "tag": "snell-in",
            "listen": "::",
            "listen_port": 443,
            "version": 6,
            "psk": "server-psk",
            "users": [
                {
                    "name": "user1",
                    "userkey": "user1-key"
                },
                {
                    "name": "user2",
                    "userkey": "user2-key"
                }
            ],
            "mode": "default"
        }
    ]
}
```

多用户模式下，顶层 `psk` 作为服务器密钥，各用户通过 `userkey` 进行认证。`name` 仅用于日志标识，非必填。

#### 生成密码

```bash
sing-box generate rand --base64 $((RANDOM % 244 + 12))
```

该命令会生成一个 12~255 字节的随机 Base64 字符串。Snell v6 要求 `psk` 长度在此范围内。

### 启动 sing-box

#### 检查配置语法

```bash
sing-box check -c /etc/sing-box/config.json
```

如果输出为空且无报错，说明配置正确。

#### 服务管理

安装完成后执行 `sudo systemctl enable sing-box && sudo systemctl restart sing-box` 启动 sing-box

| 行动 | 命令 |
|------|------|
| 启用 | `sudo systemctl enable sing-box` |
| 禁用 | `sudo systemctl disable sing-box` |
| 启动 | `sudo systemctl start sing-box` |
| 停止 | `sudo systemctl stop sing-box` |
| 强行停止 | `sudo systemctl kill sing-box` |
| 重新启动 | `sudo systemctl restart sing-box` |
| 查看日志 | `sudo journalctl -u sing-box --output cat -e` |
| 实时日志 | `sudo journalctl -u sing-box --output cat -f` |
