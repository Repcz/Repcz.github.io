## 常用 VPS 脚本

画饼中

###　检测类脚本

#### 流媒体检测

```bash
bash <(curl -L -s media.ispvps.com)
```

#### IP质量检测

```bash
bash <(curl -L -s ip.check.place)
```

#### 回程测试

```bash
curl https://raw.githubusercontent.com/ludashi2020/backtrace/main/install.sh -sSf | sh
```

#### 测速

```bash
bash <(curl -sL https://raw.githubusercontent.com/i-abc/Speedtest/main/speedtest.sh)
```

#### nexttrace 路由测试

```bash
curl -sL https://nxtrace.org/nt | bash
```

### 工具类节点

#### 人形脚本

```bash
bash <(curl -fsSL https://github.com/EAlyce/conf/raw/refs/heads/main/Linux/installTeleBox.sh)
```

#### 科技lion 多工具脚本

```bash
curl -sS -O https://raw.githubusercontent.com/kejilion/sh/main/kejilion.sh && chmod +x kejilion.sh && ./kejilion.sh
```

#### Docker watchtower

每小时自动更新 Docker 镜像，更多使用细节参考[《Watchtower：自动更新 Docker 镜像与容器》](https://www.moewah.com/archives/3863.html)

```bash
docker run -d \
  --name watchtower \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  nickfedor/watchtower \
  --cleanup \
  --interval 3600
```

#### Realm 转发

```bash
bash <(curl -sSL https://github.com/qiuxiuya/qiuxiuya/raw/refs/heads/main/VPS/realm/realm_install.sh)
```

修改 `/etc/realm/config.toml` 后 重启Realm
```bash
sudo systemctl restart realm
```

### 节点搭建脚本

#### SENLL 

* Snell 搭建、管理

```bash
wget -O snell.sh --no-check-certificate https://git.io/Snell.sh && chmod +x snell.sh && ./snell.sh
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

