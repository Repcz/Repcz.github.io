# 4. 策略(策略组)

<!-- prettier-ignore -->
!!! 注意
    以下主要讲的是 `[filter_local]`、`[filter_remote]` 或 `[policy]`  区块下的内容，所以示例都以 `[filter_local]`、`[filter_remote]` 或 `[policy]` 开头表明在其之下，并不是让你每个参数字段前都加上 `[filter_local]`、`[filter_remote]` 或 `[policy]`。

    以 `;` 或 `#` 或 `//` 开头的行为注释行。


Policy：除了内置的 [3 个策略](../quantumultx/filter.md?#31-filter)，还有以下策略：

- `Static`:「手动选择」，你需要手动选择想要的节点/策略组。
- `Available`：「故障转移」，将按顺序选择你列表中第一个可用的节点。
- `Round-Robin`：「节点轮询」，将按列表的顺序轮流使用其中的节点。
- `Dest-Hash`：「随机负载均衡」，相同域名走固定节点
- `URL-Latency-Benchmark`：「最低延迟」，选取延迟最低的节点。
- `SSID`：根据Wi-Fi的SSID参数选择策略



### 4.1 添加策略组

#### 4.1.1 `Static`:「手动选择」

`Static` 策略用于「手动选择策略」。需要手动选择想要的节点/策略组。

<!-- prettier-ignore -->
!!! 提示
    类似于 Surge 的 `Select` 策略组

需编辑配置文件，注意不要重复出现`[policy]`、`[filter_remote]`：

```
[policy]
static = Guard, reject, direct, img-url=https://raw.githubusercontent.com/Koolson/Qure/master/IconSet/Hijacking.png

[filter_remote]
# 阻止广告
https://example.com/advertising.txt, tag=🛡️ Block Advertising, force-policy=Guard, update-interval=86400, opt-parser=true, inserted-resource=true, enabled=true

# 阻止追踪
https://example.com/tracking.txt, tag=🛡️ Block Tracking, force-policy=Guard, update-interval=86400, opt-parser=true, inserted-resource=true, enabled=true

# 阻止劫持
https://example.com/hijacking.txt, tag=🛡️ Block Hijacking, force-policy=Guard, update-interval=86400, opt-parser=true, inserted-resource=true, enabled=true
```

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/Static_1.webp" width="600">


如上示例，新建了一个名为 `Guard` 的「手动选择策略组」，里面有阻止 (`REJECT`) 与直连 (`DIRECT`) 两个策略可供选择，还设置了一个策略图标。

然后添加了三条远程分流文件，分别关于广告、追踪与劫持的，强制它们的策略偏好指向 `Guard` 策略组。

这样，就可以使用一个开关管理关于广告、追踪与劫持的规则是开启 (阻止) 还是关闭 (直连)。

`img-url` 自定义图标参数是可选的，详见[节点资源图标](../quantumultx/node.md?#25)

##### UI添加

当然你也可以通过UI创建这样一个策略组


<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/Static_2.webp" width="900">

<!-- prettier-ignore -->
!!! 注意
    需要先导入配置并导入节点，否则添加策略组的按钮不会显示


`需匹配的资源标签`是节点资源 (订阅) 的名称，一般是机场名称；

`需匹配的节点标签`很好理解，就是节点的名称；

- 点击「类型」可以选择「策略组类型」
- 右上角的`<>`可以在「正则筛选」或「手动引用」之间切换
  - UI「手动引用」只能在「策略组类型」为`Static`时使用
- 图标仅支持 `144x144` 或 `108x108`，支持彩色，可以通过UI选择，也可以通过`img-url`参数编辑

<details>
  <summary> 点击查看常用正则筛选表达式</summary>


游戏节点
```
^(?=.*((?i)游戏|🎮|(\b(GAME)\b)))(?!.*((?i)回国|校园)).*$
```

回国节点
```
^(?=.*(回国))(?!.*((?i)校园|游戏|🎮|(\b(GAME)\b))).*$
```

全球节点
```
^(?=.*(.))(?!.*((?i)群|邀请|返利|循环|官网|客服|网站|网址|获取|订阅|流量|到期|机场|下次|版本|官址|备用|过期|已用|联系|邮箱|工单|贩卖|通知|倒卖|防止|国内|地址|频道|无法|说明|使用|提示|特别|访问|支持|教程|关注|更新|作者|加入|(\b(USE|USED|TOTAL|EXPIRE|EMAIL|Panel|Channel|Author)\b|(\d{4}-\d{2}-\d{2}|\dG)))).*$
```

不含港台日韩新美的节点
```
^(?=.*(.))(?!.*((?i)🇭🇰|🇹🇼|🇯🇵|🇰🇷|🇸🇬|🇺🇸|香港|台湾|日本|川日|东京|大阪|泉日|埼玉|韩国|韓|首尔|新加坡|狮|美国|波特兰|达拉斯|俄勒冈|凤凰城|费利蒙|硅谷|拉斯维加斯|洛杉矶|圣何塞|圣克拉拉|西雅图|芝加哥|群|邀请|返利|循环|官网|客服|网站|网址|获取|订阅|流量|到期|机场|下次|版本|官址|备用|过期|已用|联系|邮箱|工单|贩卖|通知|倒卖|防止|国内|地址|频道|无法|说明|使用|提示|特别|访问|支持|教程|关注|更新|作者|加入|(\b(HK|Hong|TW|Tai|Taiwan|JP|Japan|KR|Korea|SG|Singapore|US|United States|GAME|USE|USED|TOTAL|EXPIRE|EMAIL|Panel|Channel|Author)\b|(\d{4}-\d{2}-\d{2}|\dG)))).*$
```

香港节点
```
^(?=.*((?i)🇭🇰|香港|(\b(HK|Hong)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

澳门节点
```
^(?=.*((?i)🇲🇴|澳门|(\b(MO|Oman)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

台湾节点
```
^(?=.*((?i)🇹🇼|台湾|(\b(TW|Tai|Taiwan)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

日本节点
```
^(?=.*((?i)🇯🇵|日本|川日|东京|大阪|泉日|埼玉|(\b(JP|Japan)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

韩国节点

```
^(?=.*((?i)🇰🇷|韩国|韓|首尔|(\b(KR|Korea)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

新加坡节点
```
^(?=.*((?i)🇸🇬|新加坡|狮|(\b(SG|Singapore)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

美国节点
```
^(?=.*((?i)🇺🇸|美国|波特兰|达拉斯|俄勒冈|凤凰城|费利蒙|硅谷|拉斯维加斯|洛杉矶|圣何塞|圣克拉拉|西雅图|芝加哥|(\b(US|United States)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

英国节点
```
^(?=.*((?i)🇬🇧|英国|伦敦|(\b(UK|United Kingdom)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

法国节点
```
^(?=.*((?i)🇫🇷|法国|(\b(FR|France)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

德国节点
```
^(?=.*((?i)🇩🇪|德国|(\b(DE|Germany)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

加拿大节点
```
^(?=.*((?i)🇨🇦|加拿大|(\b(CA|Canada)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

俄罗斯节点
```
^(?=.*((?i)🇷🇺|俄罗斯|莫斯科|新西伯利亚|(\b(Новосиби́рская|Moscow|RU|Russia)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

印度节点
```
^(?=.*((?i)🇮🇳|印度|班加罗尔|孟买|(\b(Mumbai|IN|India)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

阿根廷节点
```
^(?=.*((?i)🇦🇷|阿根廷|(\b(AR|Argentinaia)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

土耳其节点
```
^(?=.*((?i)🇹🇷|土耳其|(\b(TR|TUR|Turkey)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

荷兰节点
```
^(?=.*((?i)🇳🇱|荷兰|(\b(NL|Holland|Netherlands)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

澳大利亚节点
```
^(?=.*((?i)🇦🇺|澳大利亚|(\b(AU|Australia)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

南非节点
```
^(?=.*((?i)🇿🇦|南非|(\b(ZA|South Africa)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

</details>

#### 4.1.2 `Available`：「故障转移」

`Available` 策略用于自动回退代理服务器。将按顺序选择你列表中第一个可用的节点。

> 类似于 Surge 的 Fallback 策略组

```
[policy]
available = Available, ProxyA, ProxyB, ProxyC

[filter_local]
final, Available
```

当 ProxyA 故障时自动切换到 ProxyB，以此类推

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/Available.webp" width="600">


#### 4.1.3 `Round-Robin`：「节点轮询」

`Round-Robin` 轮询策略用于顺序轮流使用代理服务器

```
[policy]
round-robin = RoundRobin, ProxyA, ProxyB, ProxyC

[filter_local]
final, RoundRobin
```

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/Round-Robin.webp" width="600">


#### 4.1.4 `Dest-Hash`：「随机负载均衡」

`Dest-Hash` 策略用于随机负载均衡，相同域名走固定节点，此策略对于需要会话持久性的用例特别有用。

```
[policy]
dest-hash = DestHash, ProxyA, ProxyB, ProxyC

[filter_local]
final, DestHash
```

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/Dest-Hash.webp" width="600">

#### 4.1.5 `URL-Latency-Benchmark`：「最低延迟」

`URL-Latency-Benchmark` 策略用于自动测试选出延迟最低的代理服务器


<!-- prettier-ignore -->
!!! 注意
    「延迟最低」不代表「服务器最快」，还需要考虑其他如带宽、丢包率等等

<!-- prettier-ignore -->
!!! 提示
    类似于 Surge 的 url-test 策略组

```
[policy]
url-latency-benchmark = AutoTest,  ProxyA, ProxyB, ProxyC, resource-tag-regex=^sample, server-tag-regex=^example, check-interval=600, alive-checking=false, tolerance=0

[filter_local]
final, AutoTest
```

`URL-Latency-Benchmark` 策略指向具有最佳 (评估 `tolerance` 参数，单位毫秒的) url latency 结果的服务器。

当用户手动启动 URL 测试时，其策略结果也将被更新。

此类型策略有一个名为 `check-interval` (秒) 的参数，如果此策略已由任何请求激活，则将考虑该间隔。

如果 `alive-checking` 被设置为 true，那么即使该策略处于空闲状态，也会考虑间隔时间，并启动基准测试。

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/URL-Latency-Benchmark.webp" width="600">


#### 4.1.6 `SSID`

该策略组无法通过UI添加，只能修改配置文件，其具体格式为：

```
ssid = <策略名称>, <蜂窝数据下默认策略>, <Wi-Fi 下默认策略>, <SSID 名称: 策略>`
```

```
[policy]
ssid = SSID, ProxyA, ProxyA, TP-Link:ProxyB, D-Link:ProxyC

[filter_local]
final, SSID
```

如上示例，表示在`蜂窝数据网络`及 `Wi-Fi` 下默认使用 `ProxyA` 节点，然后就是指定`特定的 SSID` 所使用的策略，当网络位于 SSID 为 `TP-Link` 时使用 `ProxyB` 节点，为 `D-Link` 时使用 `ProxyC` 节点。

再举个例子，将 `SSID` 与 `URL-Latency-Benchmark` 相结合：

```
[policy]
static=全球加速, 香港SSID, proxy

ssid=香港SSID, 香港节点, 香港节点, Wi-Fi_5G:DIRECT

url-latency-benchmark=香港节点, server-tag-regex=^(?=.*((?i)🇭🇰|香港|(\b(HK|Hong)\b)))(?!.*((?i)回国|校园|游戏|(\b(GAME)\b))).*$, check-interval=600, tolerance=0, alive-checking=false

[filter_local]
final, 全球加速
```

当 `全球加速` 选中 `香港SSID` 时：

- 特定Wi-Fi下( Wi-Fi_5G)：`全球加速` → `香港SSID` → `DIRECT`
- 非特定Wi-Fi下：`全球加速` → `香港SSID` → `香港节点` → 策略组中选择的节点
- 蜂窝数据下：`全球加速` → `香港SSID` → `香港节点` → 策略组中选择的节点

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/SSID.webp" width="900">




### 4.2 策略参数

具体参数如下，只能在配置文件中写入：

- `resource-tag-regex=` 表示使用`正则筛选`需要匹配的`资源标签`（比如机场名称）
- `server-tag-regex=` 表示使用`正则筛选`需要匹配的`节点标签`（比如节点中带有香港、美国这种）

 <!-- prettier-ignore -->
!!! 注意
    `server-tag-regex=`参数无法筛选`[server_loacl]`下的本地节点，因此推荐本地节点以「配置片段」的方式添加
  
- `check-interval=600` 表示每`600`秒检查一次节点延迟，如果此策略已由任何请求激活，则将重新计算该间隔。
- `alive-checking=false` 如果被设置为`true`，那么即使该策略处于空闲状态，也会重新计算间隔时间，并启动基准测试。
- `tolerance=0` 表示上一次节点的最低延迟数值与本次节点最低延迟数值的差值，当超过这个差值时切换至最低延迟的节点

`resource-tag-regex=` 和 `server-tag-regex=` 两者结合在一起使用，既可以筛选出只需要某服务商的特定节点用于某个策略。

`resource-tag-regex=` 和 `server-tag-regex=` 仅适用于 `static`、`available` 和 `round-robin` 类型的策略。

<details>
  <summary> 点击查看常用正则筛选表达式</summary>


游戏节点
```
^(?=.*((?i)游戏|🎮|(\b(GAME)\b)))(?!.*((?i)回国|校园)).*$
```

回国节点
```
^(?=.*(回国))(?!.*((?i)校园|游戏|🎮|(\b(GAME)\b))).*$
```

全球节点
```
^(?=.*(.))(?!.*((?i)群|邀请|返利|循环|官网|客服|网站|网址|获取|订阅|流量|到期|机场|下次|版本|官址|备用|过期|已用|联系|邮箱|工单|贩卖|通知|倒卖|防止|国内|地址|频道|无法|说明|使用|提示|特别|访问|支持|教程|关注|更新|作者|加入|(\b(USE|USED|TOTAL|EXPIRE|EMAIL|Panel|Channel|Author)\b|(\d{4}-\d{2}-\d{2}|\dG)))).*$
```

不含港台日韩新美的节点
```
^(?=.*(.))(?!.*((?i)🇭🇰|🇹🇼|🇯🇵|🇰🇷|🇸🇬|🇺🇸|香港|台湾|日本|川日|东京|大阪|泉日|埼玉|韩国|韓|首尔|新加坡|狮|美国|波特兰|达拉斯|俄勒冈|凤凰城|费利蒙|硅谷|拉斯维加斯|洛杉矶|圣何塞|圣克拉拉|西雅图|芝加哥|群|邀请|返利|循环|官网|客服|网站|网址|获取|订阅|流量|到期|机场|下次|版本|官址|备用|过期|已用|联系|邮箱|工单|贩卖|通知|倒卖|防止|国内|地址|频道|无法|说明|使用|提示|特别|访问|支持|教程|关注|更新|作者|加入|(\b(HK|Hong|TW|Tai|Taiwan|JP|Japan|KR|Korea|SG|Singapore|US|United States|GAME|USE|USED|TOTAL|EXPIRE|EMAIL|Panel|Channel|Author)\b|(\d{4}-\d{2}-\d{2}|\dG)))).*$
```

香港节点
```
^(?=.*((?i)🇭🇰|香港|(\b(HK|Hong)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

澳门节点
```
^(?=.*((?i)🇲🇴|澳门|(\b(MO|Oman)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

台湾节点
```
^(?=.*((?i)🇹🇼|台湾|(\b(TW|Tai|Taiwan)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

日本节点
```
^(?=.*((?i)🇯🇵|日本|川日|东京|大阪|泉日|埼玉|(\b(JP|Japan)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

韩国节点

```
^(?=.*((?i)🇰🇷|韩国|韓|首尔|(\b(KR|Korea)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

新加坡节点
```
^(?=.*((?i)🇸🇬|新加坡|狮|(\b(SG|Singapore)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

美国节点
```
^(?=.*((?i)🇺🇸|美国|波特兰|达拉斯|俄勒冈|凤凰城|费利蒙|硅谷|拉斯维加斯|洛杉矶|圣何塞|圣克拉拉|西雅图|芝加哥|(\b(US|United States)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

英国节点
```
^(?=.*((?i)🇬🇧|英国|伦敦|(\b(UK|United Kingdom)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

法国节点
```
^(?=.*((?i)🇫🇷|法国|(\b(FR|France)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

德国节点
```
^(?=.*((?i)🇩🇪|德国|(\b(DE|Germany)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

加拿大节点
```
^(?=.*((?i)🇨🇦|加拿大|(\b(CA|Canada)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

俄罗斯节点
```
^(?=.*((?i)🇷🇺|俄罗斯|莫斯科|新西伯利亚|(\b(Новосиби́рская|Moscow|RU|Russia)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

印度节点
```
^(?=.*((?i)🇮🇳|印度|班加罗尔|孟买|(\b(Mumbai|IN|India)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

阿根廷节点
```
^(?=.*((?i)🇦🇷|阿根廷|(\b(AR|Argentinaia)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

土耳其节点
```
^(?=.*((?i)🇹🇷|土耳其|(\b(TR|TUR|Turkey)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

荷兰节点
```
^(?=.*((?i)🇳🇱|荷兰|(\b(NL|Holland|Netherlands)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

澳大利亚节点
```
^(?=.*((?i)🇦🇺|澳大利亚|(\b(AU|Australia)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

南非节点
```
^(?=.*((?i)🇿🇦|南非|(\b(ZA|South Africa)\b)))(?!.*((?i)回国|校园|游戏|🎮|(\b(GAME)\b))).*$
```

</details>

### 4.3 修改策略组 & 删除策略组

#### 4.3.1 修改策略组

前文所述每条分流规则必须有[对应的策略](../quantumultx/filter.md?#31-filter)。

同时，QX通过UI修改策略策略组名称时，并不会同步到其他位置

因此，

修改策略组`名称`，必须通过配置文件修改，不仅要修改policy中的策略组本身，其他被引用的部分也要修改。

比如其他引用了该「需要修改策略组」的策略组、策略偏好为「需要修改策略组」的分流规则，都需进行相应的修改。

eg: 将 `香港节点` 修改为 `香港`

```
[policy]

static=手动切换, 香港节点, 美国节点, 狮城节点, 日本节点, 台湾节点, direct
url-latency-benchmark=香港节点, server-tag-regex=香港, check-interval=600, alive-checking=false, tolerance=50

[filter_local]
final, 香港节点
```

<!-- prettier-ignore -->
!!! 注意
    所有的 `香港节点` 都要修改

```
[policy]

static=手动切换, 香港, 美国节点, 狮城节点, 日本节点, 台湾节点, direct
url-latency-benchmark=香港, server-tag-regex=香港, check-interval=600, alive-checking=false, tolerance=50

[filter_local]
final, 香港
```

修改策略组`参数`

  - 比如修改 `resource-tag-regex`(需要匹配的`资源标签`) 和  `server-tag-regex`(需要匹配的`节点标签`) 或 策略组类型(仅限使用正则筛选的策略组),这些可以直接UI修改

  <img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/GroupFIX.webp" width="900">

  - 如果是修改 `check-interval`、`alive-checking`、`tolerance`，则必须配置文件修改
  - 特例：当一个`static`策略组使用 `resource-tag-regex` 或 `server-tag-regex` 参数，并同时引用了 其他的策略组，此时UI编辑将失效，必须修改配置文件
    - eg：`static=手动切换, 香港节点, direct, server-tag-regex=.`


#### 4.3.2 修改策略组图标

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/ICON.webp" width="600">

<!-- prettier-ignore -->
!!! 提示
    QX 节点资源区域的图标只支持单色，策略组区域支持彩色；
    
    图标分辨率限制144x144、108x108；

- 长按策略组/节点资源 → 图标，即可通过 UI 修改图标；
- 修改配置文件的 `img-url` 即可自定义图标
- 点此查看[图标集](../quantumultx/icon.md)


#### 4.3.3 删除策略组

如上文所述，需要删除每一处使用需要删除的策略组的地方，都需要修改。

eg: 将 `香港节点` 删除

```
[policy]

static=手动切换, 香港节点, 美国节点, 狮城节点, 日本节点, 台湾节点, direct
url-latency-benchmark=香港节点, server-tag-regex=香港, check-interval=600, alive-checking=false, tolerance=50

[filter_local]
final, 香港节点
```

 所有的 `香港节点` 都要删除，分流规则需要改为其他策略。

```
[policy]

static=手动切换, 美国节点, 狮城节点, 日本节点, 台湾节点, direct

[filter_local]
final, 美国节点
```
