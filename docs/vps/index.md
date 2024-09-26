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


### 工具类节点

#### 人形脚本


* 搭建人形(Docker)

```bash
wget https://raw.githubusercontent.com/TeamPGM/PagerMaid-Pyro/development/utils/docker.sh -O docker.sh && chmod +x docker.sh && bash docker.sh
```

* 搭建人形

```bash
wget https://raw.githubusercontent.com/TeamPGM/PagerMaid-Pyro/development/utils/install.sh -O install.sh && chmod +x install.sh && bash install.sh
```

* 搭建人形(EAlyce)

```bash
curl -k -O https://raw.githubusercontent.com/EAlyce/conf/main/PagerMaid/RXsetup.sh && chmod +x RXsetup.sh && ./RXsetup.sh
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
    containrrr/watchtower \
    --cleanup \
    -i 3600
```

### 节点搭建脚本

#### SENLL 

* Snell搭建、管理

```bash
wget -O snell.sh --no-check-certificate https://git.io/Snell.sh && chmod +x snell.sh && ./snell.sh
```

* Snell快速部署，随机端口、启动TFO，输出Surge格式节点

```bash
curl -sS -o Snell.sh https://raw.githubusercontent.com/passeway/Snell/main/Snell.sh && chmod +x Snell.sh && ./Snell.sh
```

#### Hysteria2

* Hy2快速部署，随机端口，输出Surge、Clash格式节点，不设置下载速度

```bash
curl -sS -o Hysteria.sh https://raw.githubusercontent.com/passeway/Hysteria/main/Hysteria.sh  && chmod +x Hysteria.sh && ./Hysteria.sh
```

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

#### WARP

<!-- prettier-ignore -->
!!! 注意
    这是给服务器设置 `WARP` 出站代理，不是生成WARP节点

```bash
bash <(curl -fsSL git.io/warp.sh) menu
```

#### Sing-box 多协议

```bash
wget -N -O /root/singbox.sh https://raw.githubusercontent.com/qiuxiuya/qiuxiuya/main/VPS/singbox.sh && chmod +x /root/singbox.sh && ln -sf /root/singbox.sh /usr/local/bin/singbox && bash /root/singbox.sh
```
