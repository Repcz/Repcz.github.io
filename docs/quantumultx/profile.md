# 12. 配置文件相关

### 12.1 配置文件导入

  - 下载配置:下载远程文件
  - 导入配置:从本地文件导入

<!-- prettier-ignore -->
!!! 警告
    此操作会覆盖原有配置，务必做好备份
  
<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/UI11-1.webp" width="600">
  

### 12.2 配置片段

配置片段可将本地片段写入本地文件，再作为远程文件进行引用

- [节点配置片段](../quantumultx/node.md?#222)
- [分流配置片段](../quantumultx/filter.md?i#325)
- [重写配置片段](../quantumultx/rewrite.md?#713)

### 12.3 设置关联配置

非 iCloud 云盘、非本地配置 每24小时检查更新。

关联新配置前，建议在 iCloud 云盘或 本地 创建一份当前配置的副本。

如果所关联的配置为 iCloud 云盘中的配置，那么用户对该配置所做的修改会通过 iCloud 同步到其他（如有）或当前设备。（即多个设备关联同一份配置，即可通过 iCloud 同步做到多端同步）

如果所关联的配置既不属于 iCloud 云盘 也不是 本地配置，那么用户无法在 Quantumult X 中对其进行修改。

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/UI11-2.webp" width="1500">

当存在存在多份关联配置后，可以先取消当前关联配置，再关联其他配置，即可达到 切换多配置 的操作

<img src="https://raw.githubusercontent.com/Repcz/Repcz.github.io/main/docs/quantumultx/Photo/UI11-3.webp" width="900">

