<!--
 * @Author       : wujixmm
 * @Date         : 2025-12-29 09:42:07
 * @LastEditors  : wujixmm wujixmm@gmail.com
 * @LastEditTime : 2025-12-29 15:55:03
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
  -v /$HOME/Sites/nginx/conf.d:/etc/nginx/conf.d:ro \
  -v /$HOME/Sites/nginx/log:/var/log/nginx \
  -v /$HOME/Sites/html:/usr/share/nginx/html:ro \
  nginx:latest
```
