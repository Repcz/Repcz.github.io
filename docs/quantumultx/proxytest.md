
<!-- prettier-ignore -->
!!! 注意
    - 节点延迟低不代表速度快
    - 部分机场存在劫持，此类节点建议使用不带 `generate_204` 的测试地址


#### 常用代理节点测试地址

=== "Quantumult X"
    <!-- prettier-ignore -->
    !!! 注意
        替换时请注释或删除 `[general]` 下正在使用的 `server_check_url=` 项目

    - Sukkwa 的整合性测试地址
    ```
    server_check_url=http://latency-test.skk.moe/endpoint
    ```

    - 苹果设备用于检测 Wi-Fi 是否需要认证登陆的链接
    ```
    server_check_url=http://www.apple.com/library/test/success.html
    ```

    - Google的网络联通性测试地址
    ```
    server_check_url=http://connectivitycheck.gstatic.com/generate_204
    ```

    - 微软的网络联通性测试地址
    ```
    server_check_url=http://www.msftconnecttest.com/connecttest.txt
    ```

    - 高通的联通性测试地址
    ```
    server_check_url=http://www.qualcomm.cn/generate_204
    ```

    - Cloudflare网络联通性测试地址
    ```
    server_check_url=http://cp.cloudflare.com/generate_204
    ```

    - 谷歌常用网络联通性测试地址
    ```
    server_check_url=http://www.gstatic.com/generate_204
    ```

    - 1.1.1.1网络联通性测试地址
    ```
    server_check_url=http://1.1.1.1/generate_204
    ```

=== "Loon/Surge"
    <!-- prettier-ignore -->
    !!! 注意
        替换时请注释或删除 `[General]` 下正在使用的 `proxy-test-url =` 项目

    - Sukkwa 的整合性测试地址
    ```
    proxy-test-url = http://latency-test.skk.moe/endpoint
    ```

    - 苹果设备用于检测 Wi-Fi 是否需要认证登陆的链接
    ```
    proxy-test-url = http://www.apple.com/library/test/success.html
    ```

    - Google的网络联通性测试地址
    ```
    proxy-test-url = http://connectivitycheck.gstatic.com/generate_204
    ```

    - 微软的网络联通性测试地址
    ```
    proxy-test-url = http://www.msftconnecttest.com/connecttest.txt
    ```

    - 高通的联通性测试地址
    ```
    proxy-test-url = http://www.qualcomm.cn/generate_204
    ```

    - Cloudflare网络联通性测试地址
    ```
    proxy-test-url = http://cp.cloudflare.com/generate_204
    ```

    - 谷歌常用网络联通性测试地址
    ```
    proxy-test-url = http://www.gstatic.com/generate_204
    ```

    - 1.1.1.1网络联通性测试地址
    ```
    proxy-test-url = http://1.1.1.1/generate_204
    ```

#### 常用网络联通性测试地址

=== "Quantumult X"
    <!-- prettier-ignore -->
    !!! 注意
        替换时请注释或删除 `[general]` 下正在使用的 `network_check_url=` 项目

    - VIVO
    ```
    network_check_url=http://wifi.vivo.com.cn/generate_204
    ```

    - HUAWEI
    ```
    network_check_url=http://connectivitycheck.platform.hicloud.com/generate_204
    ```

    - QUALCOMM
    ```
    network_check_url=http://www.qualcomm.cn/generate_204
    ```

    - APPLE
    ```
    network_check_url=http://www.apple.com.cn/library/test/success.html
    ```
    ```
    network_check_url=http://www.apple.com/library/test/success.html
    ```

    - XIAOMI
    ```
    network_check_url=http://http://connect.rom.miui.com/generate_204
    ```

=== "Loon/Surge"
    <!-- prettier-ignore -->
    !!! 注意
        替换时请注释或删除 `[General]` 下正在使用的 `internet-test-url = ` 项目

    - VIVO
    ```
    internet-test-url = http://wifi.vivo.com.cn/generate_204
    ```

    - HUAWEI
    ```
    internet-test-url = http://connectivitycheck.platform.hicloud.com/generate_204
    ```

    - QUALCOMM
    ```
    internet-test-url = http://www.qualcomm.cn/generate_204
    ```

    - APPLE
    ```
    internet-test-url = http://www.apple.com.cn/library/test/success.html
    ```
    ```
    internet-test-url = http://www.apple.com/library/test/success.html
    ```

    - XIAOMI
    ```
    internet-test-url = http://http://connect.rom.miui.com/generate_204
    ```


<script>
const fileList = [];
const divList = document.querySelectorAll("list item");
const githubLink = "https://github.com/clash-verge-rev/clash-verge-rev/releases";
(async () => {
  const link = "https://api.github.com/repos/clash-verge-rev/clash-verge-rev/releases/latest";
  const { assets } = await fetch(link).then((r) => r.json());
  for (const { name, browser_download_url: url } of assets) {
    fileList.push({ name, url });
  }
  for (const div of divList) {
    const logo = div.getAttribute("logo");
    const label = div.getAttribute("label");
    const keyword = div.getAttribute("keyword");
    const content = div.getAttribute("content");
    const color = div.getAttribute("color") ?? "44CC11";
    div.innerHTML = fileList.map(({ name, url }) => {
      if (name.endsWith(keyword)) {
        const a = document.createElement("a");
        a.href = url;
        const img = document.createElement("img");
        img.src = `https://img.shields.io/badge/${label}-${content}-${color}?logo=${logo}`;
        a.appendChild(img);
        return a.outerHTML;
      }
      return "";
    }).join("");
  }
})();
</script>
<style>
list{
  display: flex;
  gap: 8px;
}
</style>
