
## 节点延迟测试
=== "Quantumult X"

    <!-- prettier-ignore -->
    !!! 注意！
        节点延迟低不代表速度快；部分机场存在劫持，此类节点建议使用不带generate_204的测试地址

    #### 常用代理节点测试地址

    ⁠⁠⁠
    * 苹果设备用于检测 Wi-Fi 是否需要认证登陆的链接
    
    ```
    server_check_url=http://www.apple.com/library/test/success.html
    ```

    * Google的网络联通性测试地址

    ```
    server_check_url=http://connectivitycheck.gstatic.com/generate_204
    ```

    * 微软的网络联通性测试地址

    ```
    server_check_url=http://www.msftconnecttest.com/connecttest.txt
    ```

    * 高通的联通性测试地址

    ```
    server_check_url=http://www.qualcomm.cn/generate_204
    ```

    * Cloudflare网络联通性测试地址

    ```
    server_check_url=http://cp.cloudflare.com/generate_204
    ```
    
    * 谷歌常用网络联通性测试地址
    
    ```
    server_check_url=http://www.gstatic.com/generate_204
    ```
    
    * 1.1.1.1网络联通性测试地址
    
    ```
    server_check_url=http://1.1.1.1/generate_204
    ```


