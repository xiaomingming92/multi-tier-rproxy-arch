<!--
 * @Author       : wujixmm
 * @Date         : 2025-12-29 09:50:48
 * @LastEditors  : wujixmm wujixmm@gmail.com
 * @LastEditTime : 2025-12-29 14:41:41
 * @FilePath     : /intranetPenet/jenkens.md
 * @Description  : 
 * 
-->

# 安装步骤
- 安装官网的推荐：https://pkg.jenkins.io/debian-stable/

## 坑
- Jenkens 目前推荐的是java 21,本地如果有sdkman管理的java 不是21, 最好还是apt安装推荐版本，然后本地自启动，不走临时bash，脱离sdkman控制

## 开启自启动
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now jenkins
sudo systemctl status jenkins
```

## Jenkens 修改 webGUI 项目的 端口
```bash
sudo mkdir -p /etc/systemd/system/jenkins.service.d
sudo nano /etc/systemd/system/jenkins.service.d/override.conf

```
写入：
```ini

[Service]
Environment="JENKINS_PORT=8081"

````