# 部署 git-proxy 加速服务

项目基于 [麻婆](https://github.com/pmkol) 的开源 [git-proxy](https://github.com/pmkol/git-proxy) 加速服务

> 以下内容使用 Debian 系统，参考 ChatGPT 给出的步骤

## 部署 git-proxy

1. 下载对应平台的[二进制文件](https://github.com/pmkol/git-proxy/releases)（比如 Linux amd64）：

```bash
# 创建目录并进入
mkdir -p ~/git-proxy && cd ~/git-proxy

wget https://github.com/pmkol/git-proxy/releases/download/Prerelease/git-proxy-linux-amd64 -O git-proxy

```

2. 赋予执行权限

```bash
chmod +x git-proxy
```

3. 运行程序

```bash
./git-proxy -p 3000
```

此时 程序运行成功并占用 `3000` 端口

再 `ctrl+C` 退出程序

以下为指令：

```shell
Usage:
  git-proxy [flags]

Flags:
  -l, --bandwidth-limit int          设置带宽限制（单位：MB/s）。设为 0 表示不限制带宽。例如 --bandwidth-limit 10 表示限制总带宽为 10MB/s。
  -b, --blacklist-path string        设置仓库黑名单文件路径，用于屏蔽某些 GitHub 仓库的访问。你可以在该文件中写入如 torvalds/linux 这样的条目来禁止访问它。支持通配符 *、?。
      --deny-web-page                禁止用户访问网页请求（即浏览器访问 GitHub 页面），只允许 Git 操作。
      --deny-web-page-list strings   指定哪些网页域名应被禁止访问（配合 --deny-web-page）。可以自定义一个域名列表，例如加上 api.github.com。
      --disable-color                禁用彩色输出（终端输出变成纯色文字，适用于日志系统）。
  -d, --domain-list-path string      设置允许访问的 GitHub 域名白名单文件路径。默认包含常用 GitHub CDN 和 API 域名。
  -h, --help                         help for git-proxy
  -p, --running-port int             设置代理服务器监听的端口，默认是 30000。你可以改成其他端口，如 -p 8080。
```

## 写入 systemd 服务配置文件

1. 创建服务文件：

```bash
sudo nano /etc/systemd/system/git-proxy.service
```

写入以下内容：

```ini
[Unit]
Description=Git Proxy Server
After=network.target

[Service]
Type=simple
ExecStart=/root/git-proxy/git-proxy -p 3000
WorkingDirectory=/root/git-proxy
Restart=always
RestartSec=5
StandardOutput=file:/var/log/git-proxy.log
StandardError=file:/var/log/git-proxy-error.log
User=root

[Install]
WantedBy=multi-user.target
```

然后把上面内容复制进去，保存并退出（`Ctrl+O` → `回车` → `Ctrl+X`）。

可以根据需要修改 `ExecStart=/root/git-proxy/git-proxy -p 3000` 为你需要的指令，例如限速为 3MB/S : `ExecStart=/root/git-proxy/git-proxy -p 3000 -l -3`

2. 重新加载 systemd 并启动服务：

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable git-proxy.service
sudo systemctl start git-proxy.service
```

## 反向代理

以下仅提供 Caddy 配置，，请根据需求进修改:

```json
git.your.domain {

    # 通用重写规则：匹配一级域名开头的路径
    @domainPath path_regexp domainPath ^/([a-zA-Z0-9.-]+)(/.*)?$
    handle @domainPath {
        rewrite * /https://{re.domainPath.1}{re.domainPath.2}
        reverse_proxy 127.0.0.1:3000
    }

    # 默认反代（处理以 /https:// 开头的规范请求）
    reverse_proxy 127.0.0.1:3000
}
```

### 使用方法

参考以下格式：

`https://<your_domain>/<github_request_url>`

例如：

`https://abc.com/https://github.com/github/docs.git`