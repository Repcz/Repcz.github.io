# usque 部署全流程(WARP MASQUE · HTTP/2)

> 目标:在一台 Linux 服务器上安装 usque(WARP 客户端),开一个本地 SOCKS5 端口,
> 走 **HTTP/2 over TCP**(避开 UDP 限速),隧道内**同时支持 IPv4 + IPv6** 出口。
>
> 实测环境:香港 VPS(Sharon-NYD, Debian/root, 无原生 IPv6,靠 WARP 隧道提供 v6)

***

## 0. 原理速览

* **[usque](https://github.com/Diniboy1123/usque)** \= Go 编写的 Cloudflare WARP 客户端,基于 [MASQUE / Connect-IP(RFC 9484)](https://datatracker.ietf.org/doc/rfc9484/),支持 HTTP/3(QUIC) 与 HTTP/2(TCP) 两种隧道。
* 隧道出口是 Cloudflare 网络:出口 IP 是 CF 的 IP(v4 ≈ `104.x.x.x`, v6 ≈ `2a09:...`)。
* `--http2` \= HTTP/2 over TCP(QUIC 走 UDP,被机房限速时必用 HTTP/2)。
* 不加 `-F`/`-S` 参数 → 隧道内 IPv4 与 IPv6 **同时启用**(双栈)。
* 服务器本身没有公网 IPv6 也没关系:客户端通过隧道分配 `fd01:`/`2606:` 地址,出口走 WARP 的 v6。

***

## 1. 准备工作

* 一台 Linux 服务器(本文以 Debian/root 为例),能访问 GitHub 与 Cloudflare。
* 需要 `curl`、`wget`、`unzip`。Debian/Ubuntu 安装:
  ```bash
  apt update && apt install -y curl wget unzip
  ```
* 先确认服务器架构(决定下载哪个压缩包):
  ```bash
  uname -m
  # 输出 x86_64 → 下载 amd64
  # 输出 aarch64 → 下载 arm64
  ```

!!! 注意
    官方仅对 **Linux amd64** 做过充分测试,其他架构可能有问题。VPS 基本是 x86_64。

***

## 2. 安装程序

```bash
cd /root

# 1) 自动获取最新版本号(如 v4.2.1)与架构
VER=$(curl -s https://api.github.com/repos/Diniboy1123/usque/releases/latest | grep -o '"tag_name": *"[^"]*"' | grep -o 'v[0-9.]*')
ARCH=$(uname -m | sed 's/x86_64/amd64/; s/aarch64/arm64/')

# 2) 下载对应版本与架构的压缩包(带 UA,避免被 GitHub 拒绝)
wget --user-agent="Mozilla/5.0" "https://github.com/Diniboy1123/usque/releases/download/${VER}/usque_${VER}_linux_${ARCH}.zip" -O usque.zip

# 3) 解压,得到二进制 usque
unzip -o usque.zip -d /root/usque

# 4) 授予执行权限,并装进 PATH(后续可直接用 usque 命令)
chmod +x /root/usque/usque
install -m 755 /root/usque/usque /usr/local/bin/usque

# 5) 确认能运行
usque --help | head
```

!!! 注意
    若 `unzip` 不存在会报错,先按第 1 节安装。若无法访问 GitHub,可手动到 [Releases 页面](https://github.com/Diniboy1123/usque/releases) 下载后上传到服务器。

***

## 3. 注册 WARP 生成配置(只需一次)

```bash
cd /root/usque

# 首次注册:自动创建账号 + 注册设备 + 完成 MASQUE 登记,生成 config.json
./usque register

# 可选:指定设备名
# ./usque register -n my-warp

# 可选:已有 Warp+ 许可证时,带 license 注册
# ./usque register --license xxxxxxxxxxxxxxxx

# 检查关键字段 —— 双栈必须同时有 v4 和 v6 隧道地址
grep -E '"(ipv4|ipv6|endpoint_v4|endpoint_h2_v4)"' config.json
```

注册成功会看到 `Successful registration` 并生成 `config.json`。若提示被限流(rate-limited),等一会儿再重试即可。

!!! 注意
    `config.json` 内含 `private_key`、`access_token`、`license` 等机密,**务必妥善备份,勿泄露**。更换机器迁移时复制该文件即可,无需重新注册。

### 3.1 重新登记(enroll)

`enroll` 用于**重新登记**已有密钥:账号从 WireGuard 切换到 MASQUE、或迁移设备时使用。ZeroTrust 场景下还能刷新配置中的 IPv4/IPv6(否则 v6 可能失效)。

```bash
cd /root/usque
./usque enroll
```

!!! 警告
    `enroll` 会用 Cloudflare 服务器返回的数据**覆盖** `config.json`,执行前先备份:`cp config.json config.json.bak`。

**config.json 关键字段(示例,密钥打码):**

```json
{
  "endpoint_v4": "162.159.198.2",        // QUIC v4 端点
  "endpoint_v6": "2606:4700:103::2",     // QUIC v6 端点
  "endpoint_h2_v4": "162.159.198.2",     // HTTP/2 端点(走 TCP 443)
  "endpoint_h2_v6": "",                  // HTTP/2 v6 端点,默认留空
  "ipv4": "172.16.0.2",                  // 隧道内 IPv4 地址
  "ipv6": "2606:4700:110:xxxx:...",      // 隧道内 IPv6 地址(双栈必需)
  "private_key": "...",                  // 私钥,勿泄露
  "public_key": "...",
  "client_id": "...",
  "access_token": "...",                 // API 访问令牌,勿泄露
  "license": "..."                       // Warp+ 许可证,个人版可能为空
}
```

!!! 注意
    `endpoint_h2_v6` 可留空:HTTP/2 走 `endpoint_h2_v4`(TCP)即可,不影响 v6 出口。仅当你要让 HTTP/2 本身走 IPv6 时才需手动填写。

***

## 4. 手动试跑 + 连通性验证

```bash
cd /root/usque
./usque socks --http2 -b 127.0.0.1 -p 1080   # 前台运行,看到 "SOCKS proxy listening" 即成功
```

另开一个终端验证(用 `curl` 的 socks5h 代理,但**不要加 `-6` 参数**,`-6` 与 socks5h 不兼容,历史坑):

```bash
# IPv4 出口 → 应返回 104.x.x.x 等 CF 地址
curl -s -m 20 -x socks5h://127.0.0.1:1080 https://ipv4.icanhazip.com

# IPv6 出口 → 应返回 2a09:... 等 CF 地址
curl -s -m 20 -x socks5h://127.0.0.1:1080 https://ipv6.icanhazip.com

# 自动双栈(按目标解析走 v4 或 v6)
curl -s -m 20 -x socks5h://127.0.0.1:1080 https://api64.ipify.org
```

!!! 注意
    这里的 `-6` 是 **curl 的参数**(强制 IPv6),与 socks5h 代理不兼容。usque 自己也有一个 `-6/--ipv6`(让 MASQUE 连接走 IPv6),含义不同,服务器无原生 IPv6 时不要给 usque 加 `-6`。

***

## 5. 设置开机自启(systemd)

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

!!! 警告 "参数红线"
    `-F` 禁用隧道 IPv4、`-S` 禁用隧道 IPv6——**双栈必须两个都不加**。
    `--http2` 必须保留(服务器 UDP 被限速时 QUIC 连不上)。

!!! 提示
    想让隧道在**空闲时也保持重连**(默认仅产生出站流量后自动重连),可在 `ExecStart` 追加 `--always-reconnect`。

启用并启动:

```bash
systemctl daemon-reload
systemctl enable --now usque     # enable = 开机自启, --now = 立即启动
systemctl is-enabled usque       # 应输出 enabled
systemctl is-active usque        # 应输出 active
```

***

## 6. 日常运维命令(汇总)

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

## 7. 最终验收清单

| 检查项      | 命令                                                            | 期望                    |
| -------- | ------------------------------------------------------------- | --------------------- |
| 开机自启     | `systemctl is-enabled usque`                                  | `enabled`             |
| 运行状态     | `systemctl is-active usque`                                   | `active`              |
| HTTP/2   | `journalctl -u usque -n 20 \| grep "HTTP/2"`                  | `HTTP/2 mode enabled` |
| SOCKS 端口 | `ss -tlnp \| grep 1080`                                       | `127.0.0.1:1080` 在监听  |
| IPv4 出口  | `curl -x socks5h://127.0.0.1:1080 https://ipv4.icanhazip.com` | `104.x.x.x`           |
| IPv6 出口  | `curl -x socks5h://127.0.0.1:1080 https://ipv6.icanhazip.com` | `2a09:...`            |

***

## 8. 客户端使用

任何程序指向代理即可:

```bash
export https_proxy=socks5h://127.0.0.1:1080
export http_proxy=socks5h://127.0.0.1:1080
curl -s https://ipinfo.io/json     # 验证流量走 WARP
```

### 8.1 局域网设备使用

把 `-b 127.0.0.1` 改为 `-b ::`(v4/v6 双栈监听),并放行防火墙 1080 端口。

!!! 注意
    `-b ::` 表示双栈监听;如只想监听 IPv4 所有地址,可写 `-b 0.0.0.0`。

### 8.2 开启认证(可选)

需要认证时,官方参数是 `-u`(用户名)与 `-w`(密码),不是 `--user/--pass`:

```bash
# 前台试跑示例
./usque socks --http2 -b 0.0.0.0 -p 1080 -u myuser -w mypass
```

客户端连接时带上账号密码:

```bash
curl -x socks5://myuser:mypass@SERVER_IP:1080 https://ipinfo.io/json
```

!!! 注意
    目前只支持**一组** `user:pass`,不支持多用户。认证信息在 SOCKS5 握手阶段以明文传输,建议仅在可信内网使用。

***

## 9. 性能调优(可选)

* **增大 UDP 缓冲区**:quic-go 提示缓冲区过小时可调大(Linux):
  ```bash
  sysctl -w net.core.rmem_max=7500000
  sysctl -w net.core.wmem_max=7500000
  ```
* **更换 DNS**:默认使用 Quad9(`9.9.9.9` 等),可改用性能更好的 `1.1.1.1`:
  ```bash
  ./usque socks --http2 -d 1.1.1.1 -d 1.0.0.1 -d 2606:4700:4700::1111 -d 2606:4700:4700::1001
  ```

***

## 10. 常见坑(Known Issues)

1. **v6 测试失败但服务正常** → 别用 `curl -6 -x socks5h://...`(socks5h 与 -6 不兼容),直接访问 `https://ipv6.icanhazip.com`。
2. **QUIC 连不上/隧道反复断开** → 服务器 UDP 被限速,务必用 `--http2`。
3. **只有单栈出口** → 检查 ExecStart 是否有 `-F`(禁 v4)或 `-S`(禁 v6)。
4. **服务起来但日志不写** → usque 日志输出到 stdout,被 journald 捕获,看 `journalctl -u usque`,不是 `usque.log`。
5. **enroll 后 v6 失效** → 重新 `./usque enroll` 刷新 config 中的 IPv6 地址(ZeroTrust 场景常见)。
6. **日志出现 `H3_NO_ERROR` 断开** → 空闲过久被远端断开是**正常现象**,产生出站流量后会自动重连;如需保持在线加 `--always-reconnect`。
7. **新连接速度偏慢** → 默认使用 `reno` 拥塞控制,新连接会逐步提速,属正常现象。
8. **被防火墙识别为 WARP/QUIC** → 可用 `-s` 更换 SNI(默认 `client-masque.cloudflareclient.com`),例如 `-s zt-masque.cloudflareclient.com`。
