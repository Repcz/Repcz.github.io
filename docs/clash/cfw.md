# Clash for Windows 预处理配置使用方法

<!-- prettier-ignore -->
!!! 注意
    由于 CFW 所使用的 Clash Premium 内核并不支持解析Base64格式的订阅，因此此处仅展示在本地直接覆盖机场配置的预处理；

    CFW 本地配置请自行在仓库中自行寻找并参考上文所述方法修改对应的参数。

1.导入订阅后，确保正常链接外网，启动系统代理

![1](../clash/Photo/cfw1.webp)

2.打开CFW 设置——配置——点击预处理右侧的编辑，会弹出一个编辑器

![2](../clash/Photo/cfw2.webp)

3.点击下面按钮，复制里面的配置，在 CFW 弹出的编辑器里，替换原有内容，并保存

<a id="copyLink" href="https://raw.githubusercontent.com/Repcz/Tool/X/Clash/Premium/CFW_parser.yaml">点击自动复制文件内容</a>
<script>
  document.addEventListener('DOMContentLoaded', function () {
    const link = document.getElementById('copyLink');
    link.addEventListener('click', function (event) {
      event.preventDefault();
      const url = this.href;
      fetch(url)
        .then(response => response.text())
        .then(text => {
          const tempTextarea = document.createElement('textarea');
          tempTextarea.value = text;
          document.body.appendChild(tempTextarea);
          tempTextarea.select();
          document.execCommand('copy');
          document.body.removeChild(tempTextarea);
          alert('文件内容已复制!');
        })
        .catch(console.error);
    });
  });
</script>




![3](../clash/Photo/cfw3.webp)

4.打开CFW 配置，刷新配置，等待下载外部文件

<!-- prettier-ignore -->
!!! 注意
    CFW无法通过代理拉取资源，如果出现下载失败的错误，多试几次就好

![4](../clash/Photo/cfw4.webp)

