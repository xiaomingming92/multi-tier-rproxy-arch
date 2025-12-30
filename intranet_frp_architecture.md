<!--
 * @Author       : wujixmm
 * @Date         : 2025-12-29 19:32:26
 * @LastEditors  : wujixmm wujixmm@gmail.com
 * @LastEditTime : 2025-12-30 09:00:24
 * @FilePath     : /intranetPenet/intranet_frp_architecture.md
 * @Description  : 
 * 
-->
# 内网穿透总体架构设计（frp 多阶段演进方案）

## 架构目标
- VPS 作为公网端口调度器
- 所有公网流量经 frps 进入内网
- frpc 把公网端口映射回内网真实服务
- 支持多内网节点扩展，零重构

---
## 阶段 1：单机内网，多服务暴露

### 网络结构
```
公网用户
   │
   ▼
VPS frps (7000)
   │
   ▼
开发机 frpc
   ├─ Nginx :80
   ├─ GitLab :8080
   ├─ Jenkins :8081
   ├─ Node :3000
   ├─ Flutter :5000
   ├─ React :5173
   └─ MySQL :3306
```

### VPS 端
**/etc/frp/frps.ini**
```
[common]
bind_addr = 0.0.0.0
bind_port = 7000
token = frp-token
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = strongpassword
```

开放端口：7000, 80, 18080, 18081, 3000, 15000, 15173, 13306

### 内网 开发机
#### 端口设计
| 类型          | 本地端口 | VPS 公网端口 |
| ----------- | ---- | -------- |
| Node API    | 3000 | 3000     |
| MySQL       | 3306 | 13306    |
| Nginx Dev   | 80   | 80       |
| Jenkins     | 8081 | 18081    |
| GitLab      | 8080 | 18080    |
| Flutter Web | 5000 | 15000    |
| React Dev   | 5173 | 15173    |

---
<br/>

**/etc/frp/frpc.ini**
```
[common]
server_addr = VPS_IP
server_port = 7000
token = frp-token

[nginx]
type = tcp
local_port = 80
remote_port = 80

[gitlab]
type = tcp
local_port = 8080
remote_port = 18080

[jenkins]
type = tcp
local_port = 8081
remote_port = 18081

[node]
type = tcp
local_port = 3000
remote_port = 3000

[flutter]
type = tcp
local_port = 5000
remote_port = 15000

[react]
type = tcp
local_port = 5173
remote_port = 15173

[mysql]
type = tcp
local_port = 3306
remote_port = 13306
```

### frps / frpc 服务注册（systemd）
- VPS frps
```bash
sudo nano /etc/systemd/system/frps.service
```
写入：
```bash
[Unit]
Description=FRP Server
After=network.target

[Service]
ExecStart=/usr/local/bin/frps -c /etc/frp/frps.ini
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
执行：
````bash
sudo systemctl daemon-reload
sudo systemctl enable frps
sudo systemctl start frps
sudo systemctl status frps
```

- 内网机器 frpc
```bash
bash /etc/systemd/system/frpc.service
```
写入：
```bash
[Unit]
Description=FRP Client
After=network.target

[Service]
ExecStart=/usr/local/bin/frpc -c /etc/frp/frpc.ini
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
执行：
````bash
sudo systemctl daemon-reload
sudo systemctl enable frpc
sudo systemctl start frpc
sudo systemctl status frpc
```
- 说明

内网机器只需注册自己的 frpc 服务，systemd 管理自启动。

VPS 只需注册一次 frps 服务，管理所有内网节点连接。

每次修改 frps/frpc 配置后，sudo systemctl restart frps|frpc 即可应用。

---
## 阶段 2：多内网机器拆分
| 机器 | 服务 |
|------|------|
| 开发机 | nginx / node |
| 台式机A | gitlab |
| 台式机B | jenkins |

各机器只配置自己负责的 frpc 端口，无需修改 VPS。

---
## 阶段 3：HTTPS / 443 自动证书

VPS 使用 Caddy / Certbot 获取证书，反向代理到 frps 的 80 端口。

---
## 阶段 4：路径级统一入口
```
example.com/gitlab  → 台式机A:8080
example.com/jenkins → 台式机B:8081
```
开发机 Nginx 根据路径反向代理到对应 frp 端口。

