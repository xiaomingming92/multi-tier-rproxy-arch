<!--
 * @Author       : wujixmm
 * @Date         : 2025-12-27 09:39:08
 * @LastEditors  : wujixmm wujixmm@gmail.com
 * @LastEditTime : 2025-12-27 09:43:24
 * @FilePath     : /intranetPenet/README.md
 * @Description  : 
 * 
-->
# 内网穿透技术方案

## 软件架构
公网VPS + FRP Server + rathole 高性能隧道 + Caddy 自动 HTTPS

## 整体架构角色映射

```scss
[内网服务]
     │
  rathole client
     │  (高性能隧道)
     ▼
  rathole server  ──>  FRP server  ──>  Caddy  ──> Internet
                         │
                      FRP client

```

## 所需开源软件清单
| 角色           | 软件                       | 仓库                                                                       |
| ------------ | ------------------------ | ---------------------------------------------------------------------------- |
| 隧道引擎         | rathole                  | [https://github.com/rapiz1/rathole](https://github.com/rapiz1/rathole)     |
| 内网穿透控制层      | FRP                      | [https://github.com/fatedier/frp](https://github.com/fatedier/frp)       |
| 反向代理 / HTTPS | Caddy                    | [https://github.com/caddyserver/caddy](https://github.com/caddyserver/caddy) |
| 服务编排         | systemd / docker compose | 系统自带                                                                    |

## 各组件职责拆解
1. rathole —— 高性能稳定通道

负责：

内网 → VPS 的持久在线 TCP 隧道

特点：

Rust 实现

比 FRP 原生 tunnel 延迟低 30~50%

无控制面板，纯协议层

2. FRP —— 穿透控制层

负责：

多端口映射

服务注册、转发表

提供：

TCP / HTTP / HTTPS / UDP 穿透

统一配置入口

3. Caddy —— HTTPS 自动化边界

负责：

域名绑定

自动 ACME 证书

TLS 终止

替代 Nginx 的最大价值：

不写证书脚本

结构最小化

4. systemd / docker compose —— 部署编排

保证：

开机自启

崩溃拉起

## 配置一致性
### 部署所需文件一览
| 位置   | 文件                                  |
| ---- | ----------------------------------- |
| VPS  | rathole.toml / frps.ini / Caddyfile |
| 内网主机 | rathole.toml / frpc.ini             |
