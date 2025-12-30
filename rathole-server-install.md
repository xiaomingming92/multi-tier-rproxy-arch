<!--
 * @Author       : wujixmm
 * @Date         : 2025-12-27 10:36:41
 * @LastEditors  : wujixmm wujixmm@gmail.com
 * @LastEditTime : 2025-12-27 10:42:56
 * @FilePath     : /intranetPenet/rathole-server-install.md
 * @Description  : 
 * 
-->
# 拥有公网ip的服务器端 server 配置
Rathole Server 安装部署（Ubuntu x86_64）

适用于：腾讯云 / 阿里云 / 华为云等具备公网 IPv4 的 Ubuntu 服务器。

一、下载与安装
```bash
wget https://github.com/rapiz1/rathole/releases/latest/download/rathole-x86_64-unknown-linux-musl.zip
unzip rathole-x86_64-unknown-linux-musl.zip
chmod +x rathole
sudo mv rathole /usr/local/bin/

```
验证：
```bash
rathole --version

```

能显示版本号即正确。


二、创建 VPS 端配置
```bash
sudo mkdir -p /etc/rathole
sudo nano /etc/rathole/server.toml
```
写入内容：
```bash toml
[server]
bind_addr = "0.0.0.0:2333"
default_token = "change_me_token"

[server.services.web]
bind_addr = "0.0.0.0:18080"
```

三、放行端口（安全组）

| 协议  | 端口    |
| --- | ----- |
| TCP | 2333  |
| TCP | 18080 |

四、创建 systemd 服务
```bash
sudo nano /etc/systemd/system/rathole-server.service
```
```bash init
[Unit]
Description=Rathole Server
After=network.target

[Service]
ExecStart=/usr/local/bin/rathole /etc/rathole/server.toml
Restart=always

[Install]
WantedBy=multi-user.target
```

五、启动
sudo systemctl daemon-reexec
sudo systemctl enable rathole-server
sudo systemctl start rathole-server

六、检查监听
```bash
sudo systemctl status rathole-server
ss -lntp | grep rathole
```

应看到：

0.0.0.0:2333
0.0.0.0:18080


服务端完成。