---
date: 2024-11-15T09:34:04+08:00
title: wireguard内网穿透
description: wireguard内网穿透
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - wireguard
categories: Tutorials
---
## 简介

要实现 WireGuard 的内网穿透，通常需要至少一台公网服务器（可以是 VPS 或自建服务器）。WireGuard 在客户端和服务器之间建立加密隧道，客户端只需配置正确的密钥对和对等节点（peer）信息，即可通过公网服务器访问内网服务。内网中的设备则通过特定的端口进行转发，完成通信。
## 安装配置

#### 1. **安装 WireGuard**

首先，需要在服务器和客户端上安装 WireGuard 软件。WireGuard 可以在多种操作系统上运行，包括 Linux、Windows 和 macOS。

- **在 Ubuntu 上安装 WireGuard：**
```bash
sudo apt update
sudo apt install wireguard
```
- **在 Windows 上安装：**
  访问 [WireGuard 官方下载页面](https://www.wireguard.com/install/) 下载并安装 WireGuard。
- **在 macOS 上安装：**
  可以通过 Homebrew 安装：
```bash
brew install wireguard-tools
```

#### 2. **生成密钥对**

WireGuard 使用公私密钥对来进行身份验证。因此需要在服务器和客户端分别生成密钥对。

- **生成私钥和公钥：**
```bash
wg genkey | tee privatekey | wg pubkey > publickey
```
  这会在当前目录下生成两个文件：`privatekey`（私钥）和 `publickey`（公钥）。

- **在服务器和客户端上都需要执行此操作，生成各自的密钥对。**

#### 3. **配置服务器端 WireGuard**

在服务器上，创建一个配置文件（通常位于 `/etc/wireguard/wg0.conf`），并编辑以下内容：

```ini
[Interface]
PrivateKey = <server_private_key>      # 替换为服务器的私钥
Address = 10.0.0.1/24                  # 服务器的虚拟 IP 地址
ListenPort = 51820                      # 监听端口，确保开放

[Peer]
PublicKey = <client_public_key>         # 替换为客户端的公钥
AllowedIPs = 10.0.0.2/32                # 客户端的虚拟 IP 地址
```

- `<server_private_key>`：替换为服务器生成的私钥。
- `<client_public_key>`：替换为客户端生成的公钥。
- `Address`：为服务器分配一个虚拟 IP 地址，通常是一个私有网段（如 `10.0.0.1/24`）。
- `AllowedIPs`：指示服务器允许的客户端 IP 地址范围。

保存并退出文件。

#### 4. **配置客户端 WireGuard**

在客户端上，创建配置文件（通常位于 `/etc/wireguard/wg0.conf` 或 Windows 上的 WireGuard 客户端配置文件），并编辑以下内容：

```ini
[Interface]
PrivateKey = <client_private_key>       # 替换为客户端的私钥
Address = 10.0.0.2/24                   # 客户端的虚拟 IP 地址

[Peer]
PublicKey = <server_public_key>         # 替换为服务器的公钥
Endpoint = <server_public_ip>:51820     # 服务器的公网 IP 和监听端口
AllowedIPs = 0.0.0.0/0                  # 允许访问所有的 IP 地址
PersistentKeepalive = 25                # 每 25 秒发送一次 keepalive 包，保持连接
```

- `<client_private_key>`：替换为客户端生成的私钥。
- `<server_public_key>`：替换为服务器生成的公钥。
- `<server_public_ip>`：服务器的公网 IP 地址。
- `AllowedIPs`：设置为 `0.0.0.0/0` 表示所有流量都通过 VPN 隧道转发。如果只需要访问特定的内网地址，可以根据需要修改。

#### 5. **启动 WireGuard 服务**

完成配置后，启动 WireGuard 服务。

- **在服务器上：**
```bash
sudo wg-quick up wg0
```
  `wg0` 是可配置的 WireGuard 接口名称。如果配置正确，WireGuard 会启动并开始监听端口。

- **在客户端上：**
```bash
sudo wg-quick up wg0
```

在 Windows 上，使用 WireGuard GUI 客户端直接启动。

#### 6. **设置端口转发**

如果 WireGuard 服务器处于 NAT 后（例如在家用路由器后），需要在路由器上配置端口转发，将 WireGuard 的监听端口（例如 51820）转发到服务器的内部 IP 地址。

- 登录到路由器管理界面，找到端口转发或虚拟服务器设置。
- 添加一条端口转发规则，将外部的 51820 端口转发到 WireGuard 服务器的内网 IP 地址上的 51820 端口。

#### 7. **测试与验证**

- 在客户端设备上执行以下命令，检查是否能够 ping 通服务器和其他通过 WireGuard 隧道连接的设备：
```bash
ping 10.0.0.1
```

- 如果能成功 ping 通，表示 WireGuard 已经成功建立连接。

#### 8. **故障排除**

- **检查防火墙设置**：确保 WireGuard 所需的端口（51820）在服务器和客户端的防火墙上都已经开放。
- **检查路由配置**：如果连接不上，确认服务器的路由是否正确配置，将数据包正确转发到客户端。
- **检查端口转发**：如果服务器在 NAT 后，确保路由器正确配置了端口转发规则。

## 实际应用

以我家庭网络为例，家庭内网中在多台机器上部署了不同的服务，如下所示：
```
内网机器1： 192.168.31.100:5000 （群晖nas）
内网机器2： 192.168.31.101:5666 （ubuntu）
内网机器3： 192.168.31.102:7860 （Win10）

云服务器：145.72.0.100 （ubuntu）
```
目的是实现通过在群晖nas上和云服务器上部署wireguard服务实现内网穿透，在nas上设置端口转发来访问内网其他的机器服务。


### 1. 云服务器（145.72.0.100）上配置 WireGuard

- **安装 WireGuard**：

```bash
sudo apt update
sudo apt install wireguard
```

- **生成密钥对**：

```bash
wg genkey | tee server_private.key | wg pubkey > server_public.key
```

- **创建 WireGuard 配置文件**：

  创建 `/etc/wireguard/wg0.conf` 并填入以下内容：
  
```ini
[Interface]
Address = 10.0.0.1/24          # WireGuard 隧道的 IP
ListenPort = 51820             # WireGuard 使用的 UDP 端口
PrivateKey = <服务器私钥>        # 使用 server_private.key 中的值

# 允许转发到内网服务
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT
PostUp = iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  # eth0 替换为云服务器的网卡名
PostDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

#添加内网机器 1 的配置：
[Peer]
PublicKey = <内网机器 1 的公钥>
AllowedIPs = 10.0.0.2/32       # 内网机器 1 在 WireGuard 隧道中的 IP

#添加外网访问机器的配置
[Peer]
PublicKey = <外网访问机器 的公钥>
AllowedIPs = 10.0.0.100/32       # 外网访问机器 在 WireGuard 隧道中的 IP
```

- **启动 WireGuard**：

```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```
 
-  **注意：需要在云服务管理页面的防火墙设置中放开51820端口的udp流量。**

### 2. 在内网群晖NAS（192.168.31.100）上配置 WireGuard

- **群晖NAS安装 WireGuard**

需要通过社群套件安装，并且要ssh登录后设置root权限，编写配置文件，启动和开机自启也有些许不同。具体参照下面的链接：
[IPv4环境下群晖部署WireGuard服务端 和 客户端教程](https://post.smzdm.com/p/awzvz6r2/)

- **生成密钥对**：

```bash
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

- **创建 WireGuard 配置文件**：

  创建 `/etc/wireguard/wg0.conf` 并填入以下内容：

```ini
[Interface]
Address = 10.0.0.2/24          # 内网机器 1 在 WireGuard 隧道中的 IP
PrivateKey = <内网机器 1 的私钥>

# 启用 IP 转发
PostUp = sysctl -w net.ipv4.ip_forward=1
PostUp = iptables -t nat -A POSTROUTING -o wg0 -j MASQUERADE

#配置内网其他机器服务的端口转发
PostUp = iptables -t nat -A PREROUTING -i wg0 -p tcp --dport 5666 -j DNAT --to-destination 192.168.31.101:5666
PostUp = iptables -t nat -A PREROUTING -i wg0 -p tcp --dport 7860 -j DNAT --to-destination 192.168.31.102:7860

PostDown = iptables -t nat -D PREROUTING -i wg0 -p tcp --dport 5666 -j DNAT --to-destination 192.168.31.101:5666
PostDown = iptables -t nat -D PREROUTING -i wg0 -p tcp --dport 7860 -j DNAT --to-destination 192.168.31.102:7860

PostDown = iptables -t nat -D POSTROUTING -o wg0 -j MASQUERADE

[Peer]
PublicKey = <公网服务器的公钥>
Endpoint = 145.72.0.100:51820  # 公网服务器的 IP 和端口
AllowedIPs = 10.0.0.0/24         # 允许访问所有 IP
PersistentKeepalive = 25       # 保持连接活跃
```

**注意：当前的群晖套件在配置`AllowedIPs = 0.0.0.0/0`时会出现启动失败的情况，改成指定ip段后启动成功。**

- **启动 WireGuard 和 开机自启**：

```bash
#启动
sudo wg-quick up wg0

#配置开机自启
sudo wg-autostart enable wg0
```

### 4. 从外网机器访问服务

外网机器也需要根据其平台来配置WireGuard服务。基本类似，配置文件如下：

```ini
[Interface]
PrivateKey = <外网机器的私钥>
Address = 10.0.0.70/24 # 外网机器在 WireGuard 隧道中的 IP

[Peer]
PublicKey = <公网服务器的公钥>
AllowedIPs = 10.0.0.0/24
Endpoint = 145.72.0.100:51820
```

现在，从外网机器可以通过访问云服务器的 WireGuard 隧道 IP 进行访问。例如：

- `http://10.0.0.2:5000` 访问内网机器 1 的 5000 端口服务
- `http://10.0.0.2:5666` 访问内网机器 2 的 5666 端口服务
- `http://10.0.0.2:7860` 访问内网机器 3 的 7860 端口服务