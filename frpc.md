<!--
 * @Author       : wujixmm
 * @Date         : 2025-12-28 21:38:43
 * @LastEditors  : wujixmm wujixmm@gmail.com
 * @LastEditTime : 2025-12-29 15:40:07
 * @FilePath     : /intranetPenet/frpc.md
 * @Description  : 
 * 
-->
## 安装
frpc 安装路径
```bash
sudo mkdir -p /etc/frp
sudo cp frpc /usr/local/bin/
sudo chmod +x /usr/local/bin/frpc
```
## 配置frpc
```bash
sudo nano /etc/frp/frpc.ini
```
填入：
```ini
[common]
; server_addr = 127.0.0.1        # 连接的是本机 rathole 出口，仅供测试
server_addr = vps 公网ip # 连接的是公网ip
server_port = 4000 # vps服务端的frps服务监听端口
token = rathole-frp
tcp_mux = true
login_fail_exit = false
protocol = tcp
heartbeat_interval = 20
heartbeat_timeout = 90

# ================= HTTP / WEB 总入口 =================
[web]
type = tcp
local_ip = 127.0.0.1
local_port = 8080             # 内网 nginx 监听端口
remote_port = 18080           # VPS 公网端口

# ================= Node API =================
[node]
type = tcp
local_ip = 127.0.0.1
local_port = 3000
remote_port = 3000

# ================= Jenkins =================
[jenkins]
type = tcp
local_ip = 127.0.0.1
local_port = 8081
remote_port = 18081

# ================= GitLab =================
[gitlab]
type = tcp
local_ip = 127.0.0.1
local_port = 8929
remote_port = 18929

# ================= MySQL =================
[mysql]
type = tcp
local_ip = 127.0.0.1
local_port = 3306
remote_port = 13306


```
## 推荐端口规划
| 类型          | 本地端口 | VPS 公网端口 |
| ----------- | ---- | -------- |
| Node API    | 3000 | 3000     |
| MySQL       | 3306 | 13306    |
| Nginx Dev   | 80 | 80    |
| Jenkins     | 8081 | 18081    |
| GitLab      | 8080 | 18080    |
| Flutter Web | 5000 | 15000    |
| React Dev   | 5173 | 15173    |


## GitLab 安装坑
- 每次安装之前先确认版本
```bash
sudo rm -rf /var/lib/apt/lists/packages.gitlab.com*
sudo apt clean
sudo apt update

```
- 初始化
```bash
# 初始化配置
sudo gitlab-ctl reconfigure
````
-获取 root 初始密码
```bash
sudo cat /etc/gitlab/initial_root_password
```