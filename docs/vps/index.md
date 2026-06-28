## 常用 VPS 脚本

画饼中

###　检测类脚本

#### NodeQuality 多维度检测

```
bash <(curl -sL https://run.NodeQuality.com)
```

#### nexttrace 路由测试

```bash
curl -sL https://nxtrace.org/nt | bash
```

### 工具类节点

#### 人形脚本

[TeleBox](https://github.com/TeleBoxOrg/TeleBox)

```bash
wget https://raw.githubusercontent.com/TeleBoxOrg/TeleBox_Scripts/refs/heads/main/Install/telebox.sh -O telebox.sh && chmod +x telebox.sh && bash telebox.sh
```

#### 科技lion 多工具脚本

```bash
curl -sS -O https://raw.githubusercontent.com/kejilion/sh/main/kejilion.sh && chmod +x kejilion.sh && ./kejilion.sh
```

#### Docker watchtower

[watchtower](https://watchtower.nickfedor.com/) — Docker 容器自动更新工具。原项目 `containrrr/watchtower` 已停止维护，此 fork 由 [nicholas-fedor](https://github.com/nicholas-fedor) 持续维护。

**基础用法：** 每 3600 秒检查一次所有容器，自动更新并清理旧镜像。

```bash
docker run -d \
  --name watchtower \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  nickfedor/watchtower \
  --cleanup \
  --interval 3600
```

**仅监控不更新：** 只检查不执行更新操作，适合先观察兼容性。

```bash
docker run -d \
  --name watchtower \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  nickfedor/watchtower \
  --interval 3600 \
  --monitor-only
```

**更新后推送 Telegram 通知并附带报告：**

```bash
docker run -d \
  --name watchtower \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e WATCHTOWER_NOTIFICATION_URL="telegram://BOT_TOKEN@telegram?chats=CHAT_ID" \
  -e WATCHTOWER_NOTIFICATION_REPORT="true" \
  nickfedor/watchtower \
  --cleanup \
  --interval 3600
```

**更新报告模板示例（显示扫描数、更新数、失败数及详情）：**

```bash
docker run -d \
  --name watchtower \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e WATCHTOWER_NOTIFICATION_URL="telegram://BOT_TOKEN@telegram?chats=CHAT_ID" \
  -e WATCHTOWER_NOTIFICATION_REPORT="true" \
  -e WATCHTOWER_NOTIFICATION_TEMPLATE="{{if .Report}}{{with .Report}}{{len .Scanned}} Scanned, {{len .Updated}} Updated, {{len .Failed}} Failed{{range .Updated}} {{.Name}} ({{.ImageName}}): {{.CurrentImageID.ShortID}} → {{.LatestImageID.ShortID}}{{end}}{{range .Failed}} {{.Name}} ({{.ImageName}}): {{.Error}}{{end}}{{end}}{{end}}" \
  nickfedor/watchtower \
  --cleanup \
  --interval 3600
```

**同时推送多个渠道：**

```bash
docker run -d \
  --name watchtower \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e 'WATCHTOWER_NOTIFICATION_URL=telegram://BOT_TOKEN@telegram?chats=CHAT_ID slack://token-a/token-b/token-c' \
  -e WATCHTOWER_NOTIFICATION_REPORT="true" \
  nickfedor/watchtower \
  --cleanup \
  --interval 3600
```

**手动执行一次：** 使用 `--run-once` 立即更新所有容器，完成后容器退出。

```bash
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  nickfedor/watchtower \
  --run-once \
  --cleanup
```

**定时执行（cron 表达式）：** 替代 `--interval`，使用 6 字段 cron 格式。

```bash
docker run -d \
  --name watchtower \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  nickfedor/watchtower \
  --cleanup \
  --schedule "0 0 4 * * *"
```

<!-- prettier-ignore -->
!!! tip "通知服务 URL 参考"
    所有通知均通过 [shoutrrr](https://containrrr.dev/shoutrrr/v0.8/services/overview) 统一处理，支持以下服务格式：

    | 服务 | URL 格式 |
    |------|---------|
    | Telegram | `telegram://BOT_TOKEN@telegram?chats=CHAT_ID` |
    | Discord | `discord://WEBHOOK_ID@WEBHOOK_TOKEN` |
    | Slack | `slack://TOKEN_A/TOKEN_B/TOKEN_C` |
    | Bark | `bark://DEVICE_KEY@host` |
    | Pushover | `pushover://USER_KEY@TOKEN` |
    | Gotify | `gotify://host/TOKEN` |
    | SMTP | `smtp://USERNAME:PASSWORD@host:25?from=FROM&to=TO` |

    多个 URL 用空格分隔即可同时推送。

#### Realm 转发

[wcwq99/realm](https://github.com/wcwq99/realm)

```bash
curl -L https://raw.githubusercontent.com/wcwq98/realm/refs/heads/main/realm.sh -o realm.sh && chmod +x realm.sh && ./realm.sh
```

修改 `/root/.realm/config.toml` 后 重启 realm

```bash
sudo systemctl restart realm
```

### 节点搭建脚本

#### SENLL 

* Snell 搭建、管理

```bash
wget -O snell.sh --no-check-certificate https://git.io/Snell.sh && chmod +x snell.sh && ./snell.sh
```

* (被)开源 Snell

```bash
bash <(curl -fsSL https://s.ee/opensnell)
```

#### Hysteria2

* 官方脚本

安装/升级

```bash
bash <(curl -fsSL https://get.hy2.sh/)
```

卸载

```bash
bash <(curl -fsSL https://get.hy2.sh/) --remove
```

??? note "启动、重启、查看状态、停止 Hy2"
    启动Hysteria2

    ```
    systemctl start hysteria-server.service
    ```

    重启Hysteria2

    ```
    systemctl restart hysteria-server.service
    ```

    查看Hysteria2状态

    ```
    systemctl status hysteria-server.service
    ```

    停止Hysteria2

    ```
    systemctl stop hysteria-server.service
    ```

    设置开机自启

    ```
    systemctl enable hysteria-server.service
    ```

    查看日志
    
    ```
    journalctl -u hysteria-server.service
    ```

#### Shadowsocks(Rust)

* SS-Rust 部署、管理

```bash
wget -O ss-rust.sh --no-check-certificate https://raw.githubusercontent.com/xOS/Shadowsocks-Rust/master/ss-rust.sh && chmod +x ss-rust.sh && ./ss-rust.sh
```

