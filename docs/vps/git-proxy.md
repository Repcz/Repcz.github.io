# 部署 git-proxy 加速服务

项目基于 [麻婆](https://github.com/pmkol) 的开源 [git-proxy](https://github.com/pmkol/git-proxy) 加速服务

> 以下基于 Debian 系统，参考 ChatGPT 给出的步骤

## 部署 git-proxy

1. 下载对应平台的二进制文件（比如 Linux amd64）：

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
  -l, --bandwidth-limit int          set total bandwidth limit (MB/s), 0 as no limit
  -b, --blacklist-path string        set repository blacklist (default "blacklist.txt")
      --deny-web-page                deny web page requests
      --deny-web-page-list strings   deny web page requests list (default [github.com,gist.github.com])
      --disable-color                disable color output
  -d, --domain-list-path string      set accept domain (default "domainlist.txt")
  -h, --help                         help for git-proxy
  -p, --running-port int             disable color output (default 30000)
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