# 基于Github codespaces 免费部署云端 Windows 免费虚拟机

---

## 启动容器
 
```bash

docker compose -f win10.yml up -d

```

# GitHub Codespaces 搭建 Tailscale 出口节点完整文档

（手动后台运行 `tailscaled` 守护进程版）

---

## 一、Tailscale 介绍

### 1\. 什么是 Tailscale

Tailscale 是一款基于 **WireGuard 高性能加密协议** 构建的轻量化零信任组网工具（ZeroTrust VPN），可以让多台设备在公网上安全、点对点互联，无需公网 IP、无需端口映射、无需复杂配置，即可组建私有虚拟局域网。

### 2\. 核心组件

- **`tailscaled`**：Tailscale 后台守护进程，负责网络协商、加密隧道、路由管理、流量转发，必须常驻后台运行。

- **`tailscale`**：命令行客户端工具，用于登录、配置节点、启用出口、查看状态等操作。

### 3\. 出口节点（Exit Node）作用

出口节点是 Tailscale 网络中的**流量转发网关**。
其他设备连接该出口后，**所有公网流量会先加密转发到该节点，再由节点访问互联网**，实现借用节点 IP 访问网络的效果。

### 4\. 本方案优势

- 无需购买云服务器，使用 GitHub Codespaces 免费环境

- 手动后台运行 `tailscaled`，更稳定、可控

- 全平台通用（Windows/macOS/Linux/Android/iOS）

- 全程 WireGuard 加密，安全性高

- 部署简单、一键复制执行

---

## 二、前置准备

1. 注册并登录 [Tailscale 官网](https://tailscale.com/)

2. 进入 Tailscale 管理后台：
[https://login\.tailscale\.com/admin/machines](https://login.tailscale.com/admin/machines)

3. 配置 ACL 自动允许出口路由（推荐）
进入 **Access controls → ACL**，添加：

    ```json
    "autoApprovers": {
      "routes": {
        "0.0.0.0/0": ["你的邮箱"],
        "::/0": ["你的邮箱"]
      }
    }
    ```

---

## 三、安装 Tailscale

在 GitHub Codespaces 终端执行：

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

---

## 四、停止系统自带 tailscaled 服务

避免端口与套接字冲突：

```bash
sudo systemctl stop tailscaled
sudo systemctl disable tailscaled
sudo pkill tailscaled
```

---

## 五、创建运行目录

```bash
sudo mkdir -p /var/lib/tailscale /var/run/tailscale
sudo chmod 755 /var/lib/tailscale
```

---

## 六、手动后台启动 tailscaled 守护进程（核心）

```bash
sudo nohup tailscaled \
  --state=/var/lib/tailscale/tailscaled.state \
  --socket=/var/run/tailscale/tailscaled.sock \
  --tun=userspace-networking \
  --port=41641 \
  > /tmp/tailscaled.log 2>&1 &
```

验证是否运行：

```bash
ps aux | grep tailscaled
```

---

## 七、登录并广播为出口节点

```bash
sudo tailscale up \
  --advertise-exit-node \
  --accept-dns=false \
  --reset
```

执行后复制登录链接，在浏览器登录 Tailscale 账号完成绑定。

---

## 八、在 Tailscale 后台启用出口节点

1. 打开 [https://login\.tailscale\.com/admin/machines](https://login.tailscale.com/admin/machines)

2. 找到 `codespaces-xxx` 机器

3. 点击右侧 `⋯ → Edit route settings`

4. 勾选：

    - Use as exit node

    - \[0\.0\.0\.0/0\]\(0\.0\.0\.0/0\)

    - ::/0

5. 保存

---

## 九、防止 Codespaces 自动休眠（必做）

新开一个终端执行：

```bash
while true
do
  echo "Codespaces 保持运行 $(date)"
  sleep 30
done
```

保持该窗口不关闭即可持续在线。

---

## 十、客户端使用出口节点

### Windows / macOS / iOS / Android

1. 安装 Tailscale 并登录同一账号

2. 在节点列表找到 `codespaces-xxx`

3. 选择 **Use exit node → 该节点**

### Linux 命令行

```bash
tailscale up --exit-node=codespaces-xxx
```

---

## 十一、常用运维命令

### 查看状态

```bash
tailscale status
```

### 重启整套服务

```bash
sudo pkill tailscaled
sudo nohup tailscaled --state=/var/lib/tailscale/tailscaled.state --socket=/var/run/tailscale/tailscaled.sock --tun=userspace-networking --port=41641 > /tmp/tailscaled.log 2>&1 &
sudo tailscale up --advertise-exit-node --accept-dns=false
```

### 停止 tailscaled

```bash
sudo pkill tailscaled
```

---

## 十二、注意事项

1. GitHub Codespaces 免费版有每月使用时长限制

2. 带宽有限，不适合大流量下载、BT、高清流媒体

3. 仅可用于合法用途，违规会导致账号封禁

4. Codespaces 重启后需要重新执行启动命令

5. 出口节点依赖 `tailscaled` 后台守护进程，不可关闭

> （注：部分内容可能由 AI 生成）
