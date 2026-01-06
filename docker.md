<!--
 * @Author       : wujixmm
 * @Date         : 2025-12-29 09:42:07
 * @LastEditors  : wujixmm wujixmm@gmail.com
 * @LastEditTime : 2026-01-06 16:26:18
 * @FilePath     : /intranetPenet/docker.md
 * @Description  : 
 * 
-->

# 安装

## 更新源
sudo apt-get update

# 安装依赖
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

## 添加 Docker 官方 GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

## 添加 Docker 仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

## 安装 Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

## 启动并开机自启
sudo systemctl enable docker
sudo systemctl start docker

# 测试
docker run hello-world

## 权限问题
```bash
# 创建 docker 组（如果不存在）
sudo groupadd docker

# 当前用户加入 docker 组
sudo usermod -aG docker $USER

# 退出当前会话或重启终端，使组权限生效
newgrp docker

# 测试
docker run hello-world
docker ps -a
```

## 镜像实例运行
```bash
docker run -d \
  --name dev-nginx \
  -p 80:80 \
  -v /$HOME/Sites/nginx/nginx.conf:/etc/nginx/nginx.conf \
  -v /$HOME/Sites/nginx/conf.d:/etc/nginx/conf.d:ro \
  -v /$HOME/Sites/nginx/log:/var/log/nginx \
  -v /$HOME/Sites/html:/usr/share/nginx/html:ro \
  -v /etc/localtime:/etc/localtime:ro \
  -v /etc/timezone:/etc/timezone:ro \
  nginx:latest
```
## 镜像实例配置语法检查
```bash
docker run --rm \
  -v ~/Sites/nginx/conf.d:/etc/nginx/conf.d \
  nginx nginx -t -c /etc/nginx/nginx.conf

```

## docker 镜像源代理访问
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo nano /etc/systemd/system/docker.service.d/proxy.conf
```
写入（注意：端口号为代理的实际监听端口）:
```bash
[Service]
Environment="HTTP_PROXY=socks5://127.0.0.1:7891/"
Environment="HTTPS_PROXY=socks5://127.0.0.1:7891/"
Environment="NO_PROXY=localhost,127.0.0.1,192.168.0.0/16,172.16.0.0/12,10.0.0.0/8,*.mirror.ccs.tencentyun.com"
````
# 重新加载 systemd 配置
sudo systemctl daemon-reload
# 重启 Docker
sudo systemctl restart docker
# 验证配置是否生效
sudo systemctl show --property=Environment docker