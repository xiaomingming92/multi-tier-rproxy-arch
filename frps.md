<!--
 * @Author       : wujixmm
 * @Date         : 2025-12-29 09:36:16
 * @LastEditors  : wujixmm wujixmm@gmail.com
 * @LastEditTime : 2025-12-29 09:38:05
 * @FilePath     : /intranetPenet/frps.md
 * @Description  : 
 * 
-->

## 安装
frpc 安装路径
```bash
sudo mkdir -p /etc/frp
sudo cp frps /usr/local/bin/
sudo chmod +x /usr/local/bin/frps
```
## 配置frpc
```bash
sudo nano /etc/frp/frps.ini
```
填入：
```ini
[common]
bind_port = 4000        # FRP server 端口,frpc 那边需要填对
bind_addr = 127.0.0.1   # 只监听本机
token = rathole-frp
# vhost_http_port = 8080   # 可选，做 web 反向代理，frpc 如果有nginx, 则一般不需要

```