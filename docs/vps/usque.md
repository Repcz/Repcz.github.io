# usque 部署全流程(WARP MASQUE · HTTP/2 · 双栈 v4+v6)

> 目标:在一台 Linux 服务器上安装 usque(WARP 客户端),开一个本地 SOCKS5 端口,
> 走 **HTTP/2 over TCP**(避开 UDP 限速),隧道内**同时支持 IPv4 + IPv6** 出口。
>
> 实测环境:香港 VPS(Sharon-NYD, Debian/root, 无原生 IPv6,靠 WARP 隧道提供 v6)

***

## 0. 原理速览

* **usque** \= Go 编写的 Cloudflare WARP 客户端,支持 MASQUE / HTTP/2 / QUIC 隧道。
* 隧道出口是 Cloudflare 网络:出口 IP 是 CF 的 IP(v4 ≈ `104.x.x.x`, v6 ≈ `2a09:...`)。
* `--http2` \= HTTP/2 over TCP(QUIC 走 UDP,被机房限速时必用 HTTP/2)。
* 去掉 `-F`/`-S` 参数 → 隧道内 IPv4 与 IPv6 **同时启用**(双栈)。
* 服务器本身没有公网 IPv6 也没关系:客户端通过隧道分配 `fd01:`/`2606:` 地址,出口走 WARP 的 v6。

***

## 1. 安装程序

```bash
# 1) 下载最新 release(GitHub: Diniboy1123/usque)
#    选 linux amd64 压缩包(服务器是 x86_64 时)
cd /root
wget https://github.com/Diniboy1123/usque/releases/latest/download/usque_linux_amd64.zip -O usque.zip

# 2) 解压,得到二进制 usque
unzip -o usque.zip -d /root/usque

# 3) 授予执行权限(可移动进 PATH,如 /usr/local/bin)
chmod +x /root/usque/usque
# install -m 755 /root/usque/usque /usr/local/bin/usque   # 可选:装进 PATH

# 4) 确认能运行
/root/usque/usque --help | head
```

***

## 2. 注册 WARP 生成配置(只需一次)

```bash
cd /root/usque

# 首次注册:生成 config.json(含 private_key / ipv4 / ipv6 等)
./usque register

# 若已有 Warp+ 许可证,可带 license 注册(可选)
./usque register --license xxxxxxxxxxxxxxxx
# 已有 config 时刷新/重新注册: ./usque enroll

# 检查关键字段 —— 双栈必须同时有 v4 和 v6 隧道地址
grep -E '"(ipv4|ipv6|endpoint_v4|endpoint_h2_v4)"' config.json
```

**config.json 关键字段(示例,密钥打码):**

```json
{
  "endpoint_v4": "162.159.198.2",        // QUIC v4 端点
  "endpoint_v6": "2606:4700:103::2",     // QUIC v6 端点
  "endpoint_h2_v4": "162.159.198.2",     // HTTP/2 端点(走 TCP 443)
  "ipv4": "172.16.0.2",                  // 隧道内 IPv4 地址
  "ipv6": "2606:4700:110:xxxx:...",      // 隧道内 IPv6 地址(双栈必需)
  "private_key": "...",                  // 私钥,勿泄露
  "public_key": "...",
  "client_id": "..."
}
```

> ⚠️ `endpoint_h2_v6` 可留空:HTTP/2 走 `endpoint_h2_v4`(TCP)即可,不影响 v6 出口。

***

## 3. 手动试跑 + 连通性验证

```bash
cd /root/usque
./usque socks --http2 -b 127.0.0.1 -p 1080   # 前台运行,看到 "SOCKS proxy listening" 即成功
```

另开一个终端验证(不要用 `-6` 参数,它和 socks5h 不兼容,历史坑):

```bash
# IPv4 出口 → 应返回 104.x.x.x 等 CF 地址
curl -s -m 20 -x socks5h://127.0.0.1:1080 https://ipv4.icanhazip.com

# IPv6 出口 → 应返回 2a09:... 等 CF 地址
curl -s -m 20 -x socks5h://127.0.0.1:1080 https://ipv6.icanhazip.com

# 自动双栈(按目标解析走 v4 或 v6)
curl -s -m 20 -x socks5h://127.0.0.1:1080 https://api64.ipify.org
```

***

## 4. 设置开机自启(systemd)

创建 `/etc/systemd/system/usque.service`:

```ini
[Unit]
Description=usque WARP SOCKS5 proxy (HTTP/2 over TCP, dual-stack (v4+v6) tunnel)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
WorkingDirectory=/root/usque
ExecStart=/root/usque/usque socks --http2 -b 127.0.0.1 -p 1080
Restart=always
RestartSec=3
User=root

# 安全加固(注意:ProtectSystem=full 时 /root/usque 需可写,否则去掉或配 ReadWritePaths)
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=full
ReadWritePaths=/root/usque

[Install]
WantedBy=multi-user.target
```

> ⚠️ **参数红线**:`-F` 禁用隧道 IPv4、`-S` 禁用隧道 IPv6——**双栈必须两个都不加**。
> `--http2` 必须保留(服务器 UDP 被限速时 QUIC 连不上)。

启用并启动:

```bash
systemctl daemon-reload
systemctl enable --now usque     # enable = 开机自启, --now = 立即启动
systemctl is-enabled usque       # 应输出 enabled
systemctl is-active usque        # 应输出 active
```

***

## 5. 日常运维命令(汇总)

```bash
systemctl status usque                      # 状态
systemctl restart usque                     # 重启
journalctl -u usque -f                      # 实时日志
journalctl -u usque -n 20                   # 最近日志
# 日志关键行:
#   HTTP/2 mode enabled.
#   Using HTTP/2 endpoint 162.159.198.2:443
#   SOCKS proxy listening on 127.0.0.1:1080
#   Connected to MASQUE server
```

修改服务文件后:先备份 `cp usque.service usque.service.bak`,再 `daemon-reload && restart`。

***

## 6. 最终验收清单

| 检查项      | 命令                                                            | 期望                    |
| -------- | ------------------------------------------------------------- | --------------------- |
| 开机自启     | `systemctl is-enabled usque`                                  | `enabled`             |
| 运行状态     | `systemctl is-active usque`                                   | `active`              |
| HTTP/2   | `journalctl -u usque -n 20 \| grep "HTTP/2"`                  | `HTTP/2 mode enabled` |
| SOCKS 端口 | `ss -tlnp \| grep 1080`                                       | `127.0.0.1:1080` 在监听  |
| IPv4 出口  | `curl -x socks5h://127.0.0.1:1080 https://ipv4.icanhazip.com` | `104.x.x.x`           |
| IPv6 出口  | `curl -x socks5h://127.0.0.1:1080 https://ipv6.icanhazip.com` | `2a09:...`            |

***

## 7. 客户端使用

任何程序指向代理即可:

```bash
export https_proxy=socks5h://127.0.0.1:1080
export http_proxy=socks5h://127.0.0.1:1080
curl -s https://ipinfo.io/json     # 验证流量走 WARP
```

若要让局域网设备使用:把 `-b 127.0.0.1` 改为 `-b ::`(v4/v6 双栈监听)并放行防火墙 1080 端口;需要认证可加 `--user/--pass` 或前置 3proxy/sing-box 等。

***

## 8. 常见坑

1. **v6 测试失败但服务正常** → 别用 `curl -6 -x socks5h://...`(socks5h 与 -6 不兼容),直接访问 `https://ipv6.icanhazip.com`。
2. **QUIC 连不上/隧道反复断开** → 服务器 UDP 被限速,务必用 `--http2`。
3. **只有单栈出口** → 检查 ExecStart 是否有 `-F`(禁 v4)或 `-S`(禁 v6)。
4. **服务起来但日志不写** → usque 日志输出到 stdout,被 journald 捕获,看 `journalctl -u usque`,不是 `usque.log`。
5. **enroll 后 v6 失效** → 重新 `./usque enroll` 刷新 config 中的 IPv6 地址(ZeroTrust 场景常见)。