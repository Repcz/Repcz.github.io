# Script API

### 基本信息
- **$loon**

`$loon`: device Name, system version, app version, build version

- **$script**

`$script.name`: 被执行的脚本名称

`$script.startTime`: 执行脚本的时间

- **$config**

`$config.getConfig()`: 获取当前配置信息，返回json字符串


```javascript
{
    "running_model": 1,//运行模式，0:全局直连 1:分流模式 2:全局代理
    "all_buildin_nodes": [
        "DIRECT",
        "REJECT"
    ],
    "global_proxy": "节点选择",
    "all_policy_groups": [
        "宝贝支付",
        "奈飞影视",
        "运营劫持",
        "负载均衡",
        "全球直连",
        "国内媒体",
        "HK",
        "广告拦截",
        "漏网之鱼",
        "WiFi",
        "节点选择",
        "JP",
        "苹果服务",
        "测速",
        "健康模式",
        "BliBliArea",
        "TW",
        "谷歌服务",
        "油管视频",
        "国外网站",
        "网易解锁"
    ],
    "ssid": "loon-wifi-5g",
    "final": "节点选择",
    "policy_select": {
        "苹果服务": "全球直连",
        "广告拦截": "REJECT",
        "BliBliArea": "HK",
        "油管视频": "节点选择",
        "宝贝支付": "🇺🇲 v1|美国|A|6台负载",
        "奈飞影视": "DIRECT",
        "测速": "DIRECT",
        "谷歌服务": "健康模式",
        "国内媒体": "全球直连",
        "运营劫持": "REJECT",
        "节点选择": "JP",
        "漏网之鱼": "节点选择",
        "JP": "🇯🇵 v1|日本|A|7台负载",
        "网易解锁": "DIRECT",
        "全球直连": "DIRECT",
        "TW": "🇭🇰 v1|香港|D|7台负载|原生",
        "国外网站": "节点选择"
    }
}
```

`$config.getConfig(policyName,selectName)`: 设置policyName策略组所选策略为selectName，参数都为String，失败返回false，成功返回true

`$config.getSubPolicies(policyName, function(subPolicies){})`:获取policyName策略组的所有子策略，获取成功后回调，subPolicies为字符串子数组

`$config.getSelectedPolicy(policyName)`: 返回policyName策略组所选的子策略组名称

`$config.setRunningModel(model)`: 设置Loon当前运行模式，model类型为int，0:全局直连 1:分流模式 2:全局代理

### 本地存储
- **$persistentStore**

`$persistentStore.write(value,[key])`: 将value以key为键存储在本地，value和key类型都为字符串，key不传时为当前脚本名字的hash值，存储成功后返回true，失败返回false

`$persistentStore.read([key])`: 读取保存在本地中key映射的值，key不传时为当前脚本名字的hash值，返回相应的value，key和value都为字符串

`$persistentStore.remove()`: 清除所有使用脚本API保存在本地的数据

- **$notification**

`$notification.post(title,subtitle,content,[attach])`: 发起一个ios的本地通知，前三个参数分别为标题、副标题、通知内容，都为String类型，attach为可选内容，最为通知的附件，如通知带的一个图片\视频url或者点击通知时的触发的openurl

```javascript
//当attach为一个字符串时，表示点击通知的跳转链接
$notification.post("title","subtitle","content","loon://switch")

//如果既要支持附件和点击跳转，传入js对象
var attach = {
    "openUrl":"loon://switch",
    "mediaUrl":"https://example.com/img"
}
$notification.post("title","subtitle","content",attach)
```

### 网络请求

- **$httpClient**

`$httpClient.get(params, function(errormsg,response,data){})`:  发起一个http get请求，params是请求参数，callback是请求结束的回调

```javascript
//params为请求参数：如下
{
    url:"https://example.com/",
    timeout: 2000, //请求超时，单位ms，默认5000ms
    headers:{
        Content-Type:"application/json"
    },
    body:"{}",//仅仅在post请求中有效，格式可以是一个json对象、字符串、二进制等
    body-base64:true,//当有该字段时，会将body当做base64的格式解析成二进制，如果body参数不是base64后的二进制，请不要设定该值（build 612版本后有效）
    node:"HK - v1.0",//指定该请求使用哪一个节点或者策略组（可以使节点名称、策略组名称，也可以说是一个Loon格式的节点描述，如：node:"shadowsocksr,example.com,1070,chacha20-ietf,"password",protocol=auth_aes128_sha1,protocol-param=test,obfs=plain,obfs-param=edge.microsoft.com"）
    binary-mode:true,//请求响应返回二进制格式
}

//回调参数
errormsg: 失败原因，String类型，请求成功为null
response: js对象
{
    status:200,
    headers:{
        content-length:200
    }
}
data: String类型，响应body

```

`$httpClient.post(params, function(errormsg,response,data){})`: 发起post请求，参数、callback参数同get

`$httpClient.head(params, function(errormsg,response,data){})`: 发起head请求，参数、callback参数同get

`$httpClient.delete(params, function(errormsg,response,data){})`: 发起delete请求，参数、callback参数同get

`$httpClient.put(params, function(errormsg,response,data){})`: 发起put请求，参数、callback参数同get

`$httpClient.options(params, function(errormsg,response,data){})`: 发起options请求，参数、callback参数同get

`$httpClient.patch(params, function(errormsg,response,data){})`: 发起patch请求，参数、callback参数同get

### 其他

- **$done()**

在一般的脚本中，调用$done()表示结束脚本的执行，loon内部会进行脚本资源的释放，所以为了loon的js资源请在脚本结束时调用$done()释放资源；在http-request、http-response类型的脚本中，$done()的调用请参考相关脚本类型的说明：[Loon脚本类型](script.md)

- **$envirnoment**

仅用于generic类型的脚本中，当generic类型的脚本运用于某个节点时，`$envirnoment`对象有如下几个属性

`$environment.params.node`: 表示节点名称（build 410版本后推荐用nodeInfo）

`$environment.params.nodeInfo`: 节点简洁信息（为了安全起见，不会返回所有节点信息）