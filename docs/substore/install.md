# SubStore 部署

## iOS代理软件部署


<!-- prettier-ignore -->
!!! 注意
    iOS代理软件上部署 SubStore，依赖 MitM & 重写，使用前需**安装信任根证书**，并开启相应的开关

### 安装


- Loon：[点击一键导入](https://www.nsloon.com/openloon/import?plugin=https://raw.githubusercontent.com/sub-store-org/Sub-Store/master/config/Loon.plugin)


- QX：[点击一键导入](https://quantumult.app/x/open-app/add-resource?remote-resource=%7B%0A%20%20%22rewrite_remote%22%20%3A%20%5B%0A%20%20%20%20%22https%3A%2F%2Fmirror.ghproxy.com%2Fhttps%3A%2F%2Fraw.githubusercontent.com%2FPeng-YM%2FSub-Store%2Fmaster%2Fconfig%2FQX.snippet%2C%20tag%3DSub-Store%2C%20update-interval%3D172800%2C%20opt-parser%3Dfalse%2C%20enabled%3Dtrue%22%0A%20%20%5D%0A%7D)

- Surge & Shadowrocket

安装使用模块即可。

```
https://raw.githubusercontent.com/sub-store-org/Sub-Store/master/config/Surge-ability.sgmodule
```

- Stash

启动Stash后，在 **设置** - **配置列表** 中下拉点击 **Sub-Store** 即可。

  
### 访问

浏览器打开 https://sub.store 


## VPS Docker 部署

<!-- prettier-ignore -->
!!! 注意
    以下基于 **Debain 11** 系统

需要一台 VPS

需要一个域名，并设置 DNS解析 至 VPS 的 IP 上

　
### 设置域名解析


此处以 [CloudFlare](https://dash.cloudflare.com/) 添加 `A` 记录 为例

首先需要把 域名，托管到 CloudFlare

![dns1](Photo/dns1.webp)

![dns2](Photo/dns2.webp)

假设你的域名为`xxxxxx.xyz`，2级域名为 `sub`，则设置的域名 `sub.xxxxxx.xyz`，解析到 VPS 对应的 IP 上；

当然，则解析过去需要一定的时间。

### SSH登录

此处 SSH 客户端为 FinalShell

![ssh](Photo/ssh.webp)

独立 IP 的 VPS 一般默认 22 端口

部分机场会屏蔽 22 端口，需设置端口分流绕开此类机场

### 安装 Docker

以下将使用 [科技lion](https://kejilion.blogspot.com/2023/08/lionldnmp.html) 的脚本

```bash
curl -sS -O https://raw.githubusercontent.com/kejilion/sh/main/kejilion.sh && chmod +x kejilion.sh && ./kejilion.sh
```

1. 升级并安装 `curl`

```bash
apt update -y  && apt install -y curl
```

使用 `回车` 发送/执行 命令

![docker1](Photo/docker1.webp)

2. 执行一键脚本


```bash
curl -sS -O https://raw.githubusercontent.com/kejilion/sh/main/kejilion.sh && chmod +x kejilion.sh && ./kejilion.sh
```

3. 安装 `docker`

在脚本执行界面，输入 `6`，进入 docker 管理

![docker2](Photo/docker2.webp)

输入 `1`，安装 docker 

![docker3](Photo/docker3.webp)


等待安装完成，输入任意字符结束操作

安装完成后，SSH 界面内(不是下面的输入框)，按住 `ctrl` + `c`(这里并不是win系统里面的复制) ，结束当前脚本。

### 部署 `SubStore`

<!-- prettier-ignore -->
!!! 注意 "重要变更"
    后端版本 `2.14.376` 起使用 `SUB_STORE_BACKEND_SYNC_CRON`，旧的 `SUB_STORE_CRON` 已弃用，Docker 版将不再支持。

<!-- prettier-ignore -->
!!! 注意
    `SUB_STORE_FRONTEND_BACKEND_PATH=/` 后的字段（此处是 `2cXaAxRGfddmGz2yx1wA`）表示 API 路径，需自行设置并保存好；**不要使用特殊符号**，防止出现意外问题。

- 全功能带推送（Bark）

```bash
docker run -it -d --restart=always \
  -e "SUB_STORE_PUSH_SERVICE=https://api.day.app/XXXXXXXXXXXX/[推送标题]/[推送内容]?group=SubStore&autoCopy=1&isArchive=1&sound=shake&level=timeSensitive&icon=https%3A%2F%2Fraw.githubusercontent.com%2F58xinian%2Ficon%2Fmaster%2FSub-Store1.png" \
  -e "SUB_STORE_BACKEND_SYNC_CRON=55 23 * * *" \
  -e SUB_STORE_FRONTEND_BACKEND_PATH=/2cXaAxRGfddmGz2yx1wA \
  -p 127.0.0.1:3001:3001 \
  -v /root/sub-store-data:/opt/app/data \
  --name sub-store \
  xream/sub-store
```

- 不带推送

```bash
docker run -it -d --restart=always \
  -e "SUB_STORE_BACKEND_SYNC_CRON=55 23 * * *" \
  -e SUB_STORE_FRONTEND_BACKEND_PATH=/2cXaAxRGfddmGz2yx1wA \
  -p 127.0.0.1:3001:3001 \
  -v /root/sub-store-data:/opt/app/data \
  --name sub-store \
  xream/sub-store
```

- 自用模版（Telegram 推送）

```bash
docker run -d \
  --name sub-store \
  --restart unless-stopped \
  -p 127.0.0.1:3001:3001 \
  -v /root/sub-store-data:/opt/app/data \
  -e SUB_STORE_FRONTEND_BACKEND_PATH=/2cXaAxRGfddmGz2yx1wA \
  -e SUB_STORE_BACKEND_SYNC_CRON="30 3 * * *" \
  -e SUB_STORE_BACKEND_UPLOAD_CRON="20 3 * * *" \
  -e SUB_STORE_PUSH_SERVICE="telegram://BOT_TOKEN@telegram?chats=CHAT_ID" \
  xream/sub-store:latest
```

<!-- prettier-ignore -->
!!! info "推送服务说明"
    新版已支持 [shoutrrr](https://containrrr.dev/shoutrrr/v0.8/services/telegram) URL 格式，Telegram 示例：`telegram://BOT_TOKEN@telegram?chats=CHAT_ID`  
    也支持 Bark / PushPlus 等服务，格式为 `https://api.day.app/XXXXXXXXX/[推送标题]/[推送内容]?...`  
    或 Telegram Bot：`https://api.telegram.org/botAPI_KEY/sendMessage?chat_id=CHAT_ID&text=[推送标题][推送内容]`  
    其中 `[推送标题]` 和 `[推送内容]` 会被自动替换。

如果不知道 API 路径密码怎么生成，可以用科技 lion 的脚本：`13系统工具 → 14密码生成`

FinalShell 中，复制可以在选中后，点击按钮复制

![docker4](Photo/docker4.webp)

### 使用 `network_mode: host` 模式

<!-- prettier-ignore -->
!!! tip "适用场景"
    如需使用 IPv6 或避免端口映射冲突，可采用 `network_mode: host` 模式。

注意默认监听的是 `::`（全部接口），理论上应保证只监听 `127.0.0.1`。

可以合并端口，这样配置：

```
HOST=127.0.0.1
PORT=9876
SUB_STORE_BACKEND_API_PORT=3000
SUB_STORE_BACKEND_API_HOST=127.0.0.1
SUB_STORE_BACKEND_MERGE=true
SUB_STORE_FRONTEND_BACKEND_PATH=/2cXaAxRGfddmGz2yx1wA
```

此时仅暴露端口 `3000`，带路径访问。

或按需拆分前后端端口：

```
HOST=127.0.0.1
PORT=9876
SUB_STORE_BACKEND_API_PORT=3000
SUB_STORE_BACKEND_API_HOST=127.0.0.1
SUB_STORE_FRONTEND_PORT=3001
SUB_STORE_FRONTEND_HOST=127.0.0.1
SUB_STORE_FRONTEND_BACKEND_PATH=/2cXaAxRGfddmGz2yx1wA
```

<!-- prettier-ignore -->
!!! warning "端口说明"
    - `SUB_STORE_BACKEND_API_HOST` **永远不应暴露**，这是内部裸后端
    - `SUB_STORE_BACKEND_API_PORT` 默认为 `3000`
    - `SUB_STORE_FRONTEND_PORT` 默认为 `3001`
    - `HOST` / `PORT` 是 HTTP-META 的配置，`9876` 可能与其他服务冲突（如 `ddns-go`），可自行调整

### 更多环境变量

Sub-Store 支持通过 `.env` 文件或 `-e` 参数设置以下环境变量：

| 环境变量 | 说明 | 默认值 |
|---------|------|--------|
| `SUB_STORE_BACKEND_SYNC_CRON` | 定时同步订阅/文件到私有 Gist（替代已弃用的 `SUB_STORE_CRON`） | - |
| `SUB_STORE_BACKEND_UPLOAD_CRON` | 定时备份全部数据到 Gist | - |
| `SUB_STORE_BACKEND_DOWNLOAD_CRON` | 定时从 Gist 恢复全部数据 | - |
| `SUB_STORE_FRONTEND_BACKEND_PATH` | 前端访问后端的 API 路径前缀 | - |
| `SUB_STORE_BACKEND_MERGE` | 合并前后端端口，后端同时处理 API 和前端资源 | - |
| `SUB_STORE_BACKEND_PREFIX` | 后端（`SUB_STORE_BACKEND_API_PORT`）也加上路径前缀，防扫 | - |
| `SUB_STORE_PUSH_SERVICE` | 推送服务 URL（支持 shoutrrr / Bark / PushPlus / Telegram Bot） | - |
| `SUB_STORE_MAX_HEADER_SIZE` | 设置 undici header 大小限制（单位 bytes） | `32768` |
| `SUB_STORE_BODY_JSON_LIMIT` | 自定义 JSON Body 大小限制 | `1mb` |
| `SUB_STORE_CORS_ALLOWED_ORIGINS` | CORS 允许的域名 | `*` |
| `SUB_STORE_BACKEND_DEFAULT_PROXY` | 默认代理（SOCKS5/HTTP/HTTPS），例如 `socks5://a:b@127.0.0.1:7890` | - |
| `SUB_STORE_MMDB_COUNTRY_PATH` | MaxMind GeoLite2 Country 数据库路径 | - |
| `SUB_STORE_MMDB_ASN_PATH` | MaxMind GeoLite2 ASN 数据库路径 | - |
| `SUB_STORE_MMDB_CRON` | 定时更新 MMDB 数据库（后端 >=2.19.30） | - |
| `SUB_STORE_MMDB_COUNTRY_URL` | Country 数据库下载 URL（配合 `SUB_STORE_MMDB_CRON`） | - |
| `SUB_STORE_MMDB_ASN_URL` | ASN 数据库下载 URL（配合 `SUB_STORE_MMDB_CRON`） | - |
| `SUB_STORE_DATA_URL` | 远程数据文件链接，启动时自动拉取并恢复数据 | - |
| `SUB_STORE_DATA_URL_POST` | 拉取远程数据后执行的自定义命令，例如 `content.settings.gistToken='xxxxxxxxx'` | - |
| `SUB_STORE_BACKEND_CUSTOM_NAME` | 自定义前端显示的运行环境名称 | - |
| `SUB_STORE_BACKEND_CUSTOM_ICON` | 自定义前端显示的运行环境图标 | - |
| `SUB_STORE_X_POWERED_BY` | 自定义响应头 `X-Powered-By` | - |
| `SUB_STORE_PRODUCE_CRON` | 后台定时处理订阅（配合脚本缓存），格式：`cron,类型,名称;...` | - |

<!-- prettier-ignore -->
!!! info "`SUB_STORE_PRODUCE_CRON` 格式说明"
    格式：`cron,类型,名称` 分号连接多个。`sub` = 单条订阅，`col` = 组合订阅。
    
    示例：`0 */2 * * *,sub,a;0 */3 * * *,col,b`  
    即每 2 小时处理单条订阅 a，每 3 小时处理组合订阅 b。
    
    目的是定时处理订阅并生成脚本缓存，缓存有效期内 Surge 等 App 拉取订阅不会超时。

<!-- prettier-ignore -->
!!! info "`SUB_STORE_DATA_URL` 使用说明"
    如果要从 Gist 恢复，使用 Raw 链接 + `noCache` 参数：  
    `https://gist.githubusercontent.com/[username]/[gist_id]/raw/[filename]#noCache`  
    示例：`SUB_STORE_DATA_URL="https://gist.githubusercontent.com/username/id/raw/Sub-Store#noCache"`


### 反向代理

这里的方反向代理，简单来说就是让你从访问 `vps的ip:端口` 变成访问域名

Caddy反代 和 脚本反代，2选1即可

#### Caddy 反向代理

##### 部署 Caddy

参考[官方教程](https://caddy2.dengxiaolong.com/docs/install)，依次执行以下命令

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

##### Caddy 添加反代配置文件

粘贴以下代码，写入反代配置

!!! info ""
    注意替换`sub.xxxxx.xyz`为你的域名

```bash
cat << EOF > /etc/caddy/Caddyfile
sub.xxxxx.xyz {
    reverse_proxy 127.0.0.1:3001
    }
EOF
```

写入进程守护

```bash
sudo systemctl enable --now caddy
```

完成后重载配置

```bash
sudo systemctl reload caddy
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

#### 科技 lion 脚本反代

- 执行一键脚本

```bash
curl -sS -O https://raw.githubusercontent.com/kejilion/sh/main/kejilion.sh && chmod +x kejilion.sh && ./kejilion.sh
```

- 输入`10` → `23` 进入`站点反向代理-IP+端口`

- 安装提示依次输入对应的域名及端口即可

!!! info ""
    注意替换`sub.xxxxx.xyz`为你的域名

```
开始部署 反向代理-IP+端口
先将域名解析到本机IP: 显示你的服务器IP 
请输入你的IP或者解析过的域名: sub.xxxxx.xyz
请输入你的反代IP: 127.0.0.1
请输入你的反代端口: 3001
```


### 访问 SubStore

此时，SubStore 地址为：`https://sub.xxxxx.xyz`

其 API 为 
```
https://sub.xxxxx.xyz/2cXaAxRGfddmGz2yx1wA
```

一键配置打开前端 + 后端：
```
https://sub.xxxxx.xyz?api=https://sub.xxxxx.xyz/2cXaAxRGfddmGz2yx1wA
```

<!-- prettier-ignore -->
!!! tip "健康检查"
    访问 `https://sub.xxxxx.xyz/2cXaAxRGfddmGz2yx1wA/api/utils/env` 可查看版本信息，也可用作健康检查 URL


### 更新 Sub-Store

和其他 Docker 服务一样，可使用 [watchtower](https://watchtower.nickfedor.com/) 自动更新（原项目 `containrrr/watchtower` 已停止维护，此 fork 由 [nicholas-fedor](https://github.com/nicholas-fedor) 持续维护）：

**基础自动更新：** 每 3600 秒检查 Sub-Store 镜像，自动更新并清理旧版本。

```bash
docker run -d \
  --name watchtower \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  nickfedor/watchtower \
  --interval 3600 \
  --cleanup \
  sub-store
```

**更新后推送 Telegram 通知：** 配合 [shoutrrr](https://containrrr.dev/shoutrrr/v0.8/services/overview) 支持的任意服务。

```bash
docker run -d \
  --name watchtower \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e WATCHTOWER_NOTIFICATION_URL="telegram://BOT_TOKEN@telegram?chats=CHAT_ID" \
  -e WATCHTOWER_NOTIFICATION_REPORT="true" \
  nickfedor/watchtower \
  --interval 3600 \
  --cleanup \
  sub-store
```

<!-- prettier-ignore -->
!!! tip "通知服务 URL 参考"
    | 服务 | URL 格式 |
    |------|---------|
    | Telegram | `telegram://BOT_TOKEN@telegram?chats=CHAT_ID` |
    | Discord | `discord://WEBHOOK_ID@WEBHOOK_TOKEN` |
    | Slack | `slack://TOKEN_A/TOKEN_B/TOKEN_C` |
    | Bark | `bark://DEVICE_KEY@host` |
    | Pushover | `pushover://USER_KEY@TOKEN` |
    | Gotify | `gotify://host/TOKEN` |
    | SMTP | `smtp://USERNAME:PASSWORD@host:25?from=FROM&to=TO` |

### 查看日志

```bash
docker logs -f -t --tail 100 sub-store
```


### 备份注意事项

<!-- prettier-ignore -->
!!! warning "GitHub Token 安全"
    GitHub 会扫描明文 Gist 中的 Token。Sub-Store 备份时：
    
    - 选择**明文**备份 → 不会备份已设置的 GitHub Token
    - 选择 **Base64 编码**备份 → 完整备份（默认）
    
    如需明文备份可通过 API：`/api/utils/backup?action=upload&encode=plaintext`  
    保留现有 Token 恢复：`/api/utils/backup?action=download&keep=settings.gistToken`

<!-- prettier-ignore -->
!!! danger "Token 被吊销"
    近期备份失败的用户可能是 Token 被 GitHub 扫描后吊销。解决方法：
    
    1. 更新 Sub-Store 到最新版本
    2. 生成新的 GitHub Token
    3. 删除旧的 Gist
    4. 保存新 Token 后重新上传备份


## HTTP-META

带 `http-meta` tag 的镜像（`xream/sub-store:latest-http-meta`）包含 [HTTP-META](https://github.com/xream/http-meta)。

可直接使用需要 HTTP-META 功能的脚本，本地端口号为默认值，无需额外设置。

若需要自定义 HTTP-META 的端口号，使用环境变量 `PORT=9876`。

进行调试时，可设置：

```bash
-e META_DISABLE_AUTO_CLEAN=true \
-e META_TEMP_FOLDER=/opt/app/data
```

这样能查看每次运行时核心的日志和配置。

相关资源：

- [Sub-Store 教程脚本合集](https://t.me/zhetengsha/214)
- [节点测活脚本](https://t.me/zhetengsha/1210)
- [GPT 检测脚本](https://t.me/zhetengsha/1209)
- [入口 & 落地检测完整示例](https://t.me/zhetengsha/1415)
