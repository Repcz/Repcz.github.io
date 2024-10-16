# 8. HTTP 请求


「HTTP 请求」所执行的功能，可简单的理解为执行脚本

脚本分为 「CRON 定时脚本任务」、「UI交互脚本」、「网络切换脚本」

其对应的配置文件位置为 `[task_local]`

可通过首页下方工具栏进入（默认UI），或　点击右下角「风车」进入设置 → 下方「工具 & 分析」- 「HTTP 请求」

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/UI12-1.webp" width="600">

### 8.1 添加 HTTP 请求

<!-- prettier-ignore -->
!!! 注意
    以下主要讲的是 `[task_local]`  区块下的内容，所以示例都以 `[task_local]` 开头表明在其之下，并不是让你每个参数字段前都加上 `[task_local]`。


#### 8.1.1 配置文本添加

```
[task_local]
event-network https://raw.githubusercontent.com/xream/scripts/main/surge/modules/network-info/net-lsp-x.js, tag=网络信息变化 𝕏, img-url=https://raw.githubusercontent.com/Koolson/Qure/master/IconSet/Color/Global.webp, enabled=true
```

```
[task_local]
0 0 1 1 * https://github.com/sub-store-org/Sub-Store/releases/latest/download/cron-sync-artifacts.min.js, tag=SubStore同步, img-url=https://raw.githubusercontent.com/fmz200/wool_scripts/main/icons/apps/SubStore.webp, enabled=true
```

```
[task_local]
event-interaction https://raw.githubusercontent.com/xream/scripts/main/surge/modules/network-info/net-lsp-x.js, tag=网络信息 𝕏, img-url=https://raw.githubusercontent.com/Koolson/Qure/master/IconSet/Color/Global.webp, enabled=true
```

其基本格式为

```
<脚本参数>, <资源链接>, <资源标签>, <资源图标>, <是否启用>
```

脚本参数:

  - `event-network`：对应「网络切换脚本」，当网络发生变化时自动执行

  - `event-interaction`：对应「UI交互脚本」，须与UI交互执行

  - `5或6位CRON表达式`：对应「CRON 定时脚本任务」，5位表达式从分开始，具体请自行谷歌

资源链接：脚本资源位置，可为远程脚本，也可为本地脚本

`enabled`：是否启用该 HTTP 请求，若不使用可改为 `false`；

<!-- prettier-ignore -->
!!! 注意
    脚本任务需 Quantumult X Tunnel （VPN） 处于运行状态，以及计划任务开关(右上角⏰)为开启状态

#### 8.1.2 UI 添加

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/UI12-2.webp" >

- 如果给的是「完整的任务格式」，如：
  
```
event-network https://raw.githubusercontent.com/xream/scripts/main/surge/modules/network-info/net-lsp-x.js, tag=网络信息变化 𝕏, img-url=https://raw.githubusercontent.com/Koolson/Qure/master/IconSet/Color/Global.webp, enabled=true
```

  可通过右上角 `＋`→ `文本方式添加`，直接粘贴即可 

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/UI12-3.webp" width="600">

- 如果给的是「定时任务类型」，且为单独的脚本链接(或本地脚本文件)，请阅读脚本内容，根据其需要，设置 CRON表达式

  可通过右上角 `＋`→ `高级`，填写参数即可 

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/UI12-4.webp" width="600">

#### 8.2 执行脚本

<!-- prettier-ignore -->
!!! 注意
    脚本任务需 Quantumult X Tunnel （VPN） 处于运行状态，以及计划任务开关(右上角⏰)为开启状态

- `event-network`：对应「网络切换脚本」，当网络发生变化时自动执行


- `event-interaction`：对应「UI交互脚本」，须与UI交互执行

  启动 VPN 后，长按节点，即可执行

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/UI12-5.webp" width="600">


- `5或6位CRON表达式`：对应「CRON 定时脚本任务」，5位表达式从分开始，具体请自行谷歌。也可左滑点击按钮执行或查看执行时的日志

#### 8.3 任务仓库

- 添加任务仓库

<details>
  <summary> 路径需要填写资源的raw格式链接，点此查看教程</summary>

以下方的链接举例(这是个网页，不是真正能使用的资源链接)：

```
https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/QuantumultX/12306/12306.list
```

例如在末尾添加`?raw=ture`：

```
https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/QuantumultX/12306/12306.list?raw=ture
```

或者直接点击`raw`或者`view`，⁠使用跳转后的链接：

```
https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/QuantumultX/12306/12306.list
```

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/raw1.webp" >

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/raw2.webp" >



或者将链接里的`blob`⁠修改为`raw`：

```
https://github.com/blackmatrix7/ios_rule_script/raw/master/rule/QuantumultX/12306/12306.list
```



</details>


<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/UI12-6.webp" width="600">

- 从仓库中添加任务

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/UI12-7.webp" width="600">

