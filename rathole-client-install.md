<!--
 * @Author       : wujixmm
 * @Date         : 2025-12-27 10:43:42
 * @LastEditors  : wujixmm wujixmm@gmail.com
 * @LastEditTime : 2025-12-29 09:16:45
 * @FilePath     : /intranetPenet/rathole-client-install.md
 * @Description  : 
 * 
-->
Rathole Client 安装部署（Ubuntu x86_64）

适用于：需要将内网服务映射到公网 VPS 的内网主机。

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

二、创建配置文件
```bash
sudo mkdir -p /etc/rathole
sudo nano /etc/rathole/client.toml
```

写入示例：
```bash toml
[client]
remote_addr = "VPS公网IP:2333"
default_token = "change_me_token"

[client.services.web]
local_addr = "127.0.0.1:8080" # 可选

```
说明：

- remote_addr：VPS 的公网 IP + 2333 端口
- default_token：与 server.toml 中 default_token 一致
-local_addr：内网要映射的服务地址，例如本地 web 服务 127.0.0.1:8080

三、创建 systemd 服务
```bash 
sudo nano /etc/systemd/system/rathole-client.service
```
写入：
``` ini
[Unit]
Description=Rathole Client
After=network.target

[Service]
ExecStart=/usr/local/bin/rathole /etc/rathole/client.toml
Restart=always

[Install]
WantedBy=multi-user.target
```

四、启动服务
```bash
sudo systemctl daemon-reexec
sudo systemctl enable rathole-client
sudo systemctl start rathole-client
```

五、检查状态
```bash
sudo systemctl status rathole-client

ss -tnp | grep rathole
```

如果成功，内网服务端口会通过 VPS 公开映射，外部访问：

http://VPS公网IP:18080


即可访问内网服务。

客户端部署完成，可与服务端形成完整内网穿透通道。