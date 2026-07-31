# 7. 插件

> 以下搬运至 [Loon官方文档 ](https://loon0x00.github.io/LoonManual/#/loon/plugin)，不定时更新


插件是规则、复写、脚本的集合，相当于一个子配置，常常用来代表一个扩展功能。

<!-- prettier-ignore -->
!!! 提示
    此区域对应「配置标签页」-「插件」区域 - `插件`


点击顶部搜索栏可对 Name、Author、Describe 进行搜索

- 点击右上角🔄，可更新

- 拖动右上角 `≡`，可进行排序

- 点击右上角 `＋`，可添加插件

点击插件可查看详情，部分插件可对参数进行调整

![1.7](Photo/1.7.webp){: width=900}


### 7.1 插件可包含的配置模块

#### 完整示例

```
#!name = 示例插件
#!desc = 展示插件信息和用户参数
#!author = Loon
#!homepage = https://example.com
#!icon = https://example.com/icon.png
#!system = iOS,iPadOS,tvOS,macOS
#!system_version = 15
#!loon_version = 3.5.1(978)
#!tag = 示例,工具
#!type = normal

[Argument]
name = input,"Loon",tag=名称,desc=输入一个名称
region = select,"CN","US","JP",tag=地区
enabled = switch,true,tag=启用

[General]
bypass-tun =
skip-proxy =
real-ip =
dns-server =

[Rule]

[Rewrite]

[Host]

[Script]
http-response ^https?:\/\/example\.com\/conf\/server-mapping script-path=remove_ads.js,requires-body=true,tag=移除广告,argument=[{name},{region},{enabled}]

[Mitm]
hostname = example.com
```

#### 插件信息

以 `#!` 开头的字段用于描述插件：

| 字段 | 说明 |
|---|---|
| `#!name` | 插件名称 |
| `#!desc` | 功能说明 |
| `#!author` | 作者 |
| `#!homepage` | 主页地址 |
| `#!icon` | 图标地址 |
| `#!system` | 支持的系统，不区分大小写；未填写表示全部支持 |
| `#!system_version` | 最低系统版本，如 `15.0` |
| `#!loon_version` | 最低 Loon 版本，如 `3.5.1(978)` |
| `#!tag` | 分类标签 |
| `#!type` | 插件类型 |

Loon 3.5.0 (969) 支持以下插件类型：

- `normal`：普通插件。
- `parser`：资源解析器，可在节点、规则和配置订阅页面中选择。

#### 插件参数 `[Argument]`

<!-- prettier-ignore -->
!!! 注意
    适用于 Build 733 及以上版本。该模块声明需要用户填写或选择的参数，Loon 会自动生成对应界面。

<!-- prettier-ignore -->
!!! 提示
    早期插件使用 `#!input = 参数名` / `#!select = 参数名,可选值1,可选值2` 声明参数，Loon 仍兼容该旧语法以加载旧插件，但新插件请使用 `[Argument]` 模块。

基本格式：

```text
参数名 = 控件类型,默认值或可选值,tag=标题,desc=说明
```

支持的控件：

| 类型 | 说明 |
|---|---|
| `input` | 文本输入；默认值可省略 |
| `select` | 单选列表；第一个值为默认值 |
| `switch` | 开关；默认值为 `false` |

```
[Argument]
name = input,"Loon",tag=名称
region = select,"CN","US","JP",tag=地区
enabled = switch,true,tag=启用
```

在脚本中使用 `$argument.name`、`$argument.region` 和 `$argument.enabled` 读取参数：

```
[Script]
http-request ^https:\/\/example\.com script-path=request.js,argument=[{name},{region},{enabled}]
```

参数也可以用于 Cron 表达式；`switch` 参数可以控制脚本是否启用：

```
[Script]
cron {cronExpression} script-path=task.js,timeout=300,tag=自动运行
http-request ^https:\/\/example\.com script-path=request.js,enable={enabled}
```

在新版 Rewrite 中通过 `${参数名}` 引用插件参数。

### 7.2 插件中规则的策略

插件内的规则指向的策略只能有如下三种，当规则不指定策略时，会默认使用DIRECT

1. `DIRECT`：流量不经过任何节点，直接发送到目标地址
2. [`REJECT`](policy.md#422)：不将流量发送到任何服务器，一般用于去广告
3. `PROXY`：代表用户在进行插件配置时手动选择的策略组。如果用户指定了PROXY，但插件却没有进行配置，那最终将按照无法找到策略组的逻辑进行处理（即使用App全局模式下全局策略中第一个节点）

### 7.3 配置文件添加插件

<!-- prettier-ignore -->
!!! 注意
    以下主要讲的是 `[Plugin]` 区块下的内容，所以示例都以 `[Plugin]` 开头表明在其之下，并不是让你每个参数字段前都加上 `[Plugin]`。


```
[Plugin]
https://gitlab.com/lodepuly/vpn_tool/-/raw/master/Tool/Loon/Plugin/LoonGallery.plugin, policy=手动切换, enabled=true

```

`<插件资源链接>, <策略偏好>, <是否启用>`

- `policy= `<策略偏好>：取决于插件内是否存在相关字段
- `enabled =` <是否启用>: 若不使用可改为 `false`


### 7.4 插件推荐

- [可莉插件大全](https://hub.kelee.one/)
