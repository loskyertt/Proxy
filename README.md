# Mihomo (Clash Meta) 代理配置

为 [mihomo](https://github.com/MetaCubeX/mihomo) 内核量身定制的全能代理配置文件，基于 TUN 模式实现全流量接管，附带精选 WebUI 面板和详细部署指南。

## 目录

- [文件说明](#1-文件说明)
- [技术栈](#2-技术栈)
- [功能特性](#3-功能特性)
- [使用方式](#4-使用方式)
- [配置文件详解](#5-配置文件详解)
- [自定义配置](#6-自定义配置)
- [故障排除](#7-故障排除)

---

# 1. 文件说明

```
Proxy/
├── config.yaml           # 主配置文件（生产用，已脱敏订阅链接）
├── metacubexd/           # WebUI 面板 - MetaCubeX/metacubexd
├── zashboard/            # WebUI 面板 - Zephyruso/zashboard
├── .gitignore
├── LICENSE               # MIT License
└── README.md
```

下载链接：

- [mihomo 内核 - MetaCubeX/mihomo](https://github.com/MetaCubeX/mihomo)
- [Dashboard - Zephyruso/zashboard](https://github.com/Zephyruso/zashboard)
- [Dashboard - MetaCubeX/metacubexd](https://github.com/MetaCubeX/metacubexd)

# 2. 技术栈

- **代理内核**：[mihomo](https://github.com/MetaCubeX/mihomo)（Clash Meta，Go 语言实现）
- **配置格式**：YAML
- **代理协议**：VMess / VLESS / Shadowsocks / Trojan / Hysteria2 / TUIC 等（由订阅提供）
- **路由模式**：Rule（规则路由）+ TUN（虚拟网卡）
- **DNS 模式**：Fake-IP + 分流
- **Geo 资源**：GeoIP / GeoSite（自动更新）
- **WebUI**：metacubexd（Nuxt）/ zashboard（Vue）
- **运行平台**：Linux（Arch Linux / systemd）

# 3. 功能特性

- **TUN 全流量接管** — 无需逐个应用配置代理，系统全局透明代理
- **Fake-IP DNS** — 减少 DNS 查询延迟，提升访问速度
- **智能分流** — 国内外流量自动分离（直连/代理/拒绝）
- **精细化服务路由** — OpenAI / Claude / GitHub / Netflix / Disney+ / YouTube / Spotify / TikTok / Telegram / Twitter / Copilot / OneDrive 等独立代理组
- **流量嗅探** — 自动识别 TLS / HTTP / QUIC 流量，精准匹配域名规则
- **Proxy Provider** — 支持多订阅源自动更新，健康检查
- **Rule Provider** — 规则集远程自动更新（Loyalsoldier 规则）
- **Geo 数据自动更新** — GeoIP / GeoSite / ASN 每日自动同步
- **双 WebUI** — 同时提供 metacubexd 和 zashboard 两种面板

# 4. 使用方式

> 这里以 Arch Linux 为例。

## 4.1 安装和基本配置

### 安装 mihomo

```bash
sudo pacman -S mihomo
```

> [!note]
> mihomo 只有在 [archlinuxcn](https://help.mirror.nju.edu.cn/archlinuxcn/) 源、[chaotic-aur](https://aur.chaotic.cx/) 源、aur 源中才有。没有网络代理的化，建议先添加 [archlinuxcn](https://help.mirror.nju.edu.cn/archlinuxcn/) 源后，再进行下载安装。

### 配置 WebUI

WebUI 只是静态文件，你可以用本项目的 `ui` 或者从 [GitHub MetaCubeX/metacubexd](https://github.com/MetaCubeX/metacubexd?tab=readme-ov-file) 下载。还可以通过 pacman 安装：`sudo pacman -S metacubexd-bin`（archlinuxcn 源和 aur 源才有 MetaCubexXD）。

### 启动服务

```bash
sudo systemctl enable --now mihomo.service
```

### 部署配置和 UI

> 配置文件位置是 `/etc/mihomo/config.yaml` 或 `~/.config/mihomo/config.yaml`。一般是将 `config.yaml` 复制到 `/etc/mihomo/config.yaml` 下，而 `ui` 目录是放在 `/var/lib/mihomo/` 目录下。

复制配置文件：

```bash
sudo cp config.yaml /etc/mihomo/config.yaml
```

> **重要**：务必在 `config.yaml` 的 `proxy-providers` 部分填入你的订阅链接。

复制 `ui` 文件（任选一个 dashboard 目录即可）：

```bash
sudo cp -r zashboard /var/lib/mihomo/ui
```

### 重启服务

```bash
sudo systemctl restart mihomo.service
```

> 一般情况下，将配置文件和 `ui` 文件复制到相应目录后，再重启 mihomo.service 就能成功使用了。如果没有成功，继续看下文的问题解决部分。

## 4.2 验证运行状态

```bash
# 查看服务状态
sudo systemctl status mihomo.service

# 查看日志
sudo journalctl -fu mihomo.service

# 查看 TUN 网卡是否创建
ip a show Meta
```

打开浏览器访问 `http://127.0.0.1:9090/ui` 进入 WebUI 管理面板。

## 4.3 常用操作速查

| 命令 | 说明 |
|------|------|
| `sudo systemctl start mihomo` | 启动服务 |
| `sudo systemctl stop mihomo` | 停止服务 |
| `sudo systemctl restart mihomo` | 重启服务 |
| `sudo systemctl enable --now mihomo` | 开机自启并立即启动 |
| `sudo journalctl -fu mihomo` | 实时查看日志 |
| `sudo mihomo -t -d /etc/mihomo` | 测试配置是否正确 |
| `sudo mihomo -d /etc/mihomo` | 前台运行（调试用） |

# 5. 配置文件详解

## 5.1 配置架构概览

```yaml
# 主配置结构
config.yaml
├── 全局配置（端口/模式/日志）
├── TUN 配置（虚拟网卡/路由/DNS 劫持）
├── DNS 配置（Fake-IP/分流/上游 DNS）
├── 流量嗅探（TLS/HTTP/QUIC）
├── 代理组（Auto / PROXY / OpenAI / Claude / ...）
├── 代理集合（Proxy Providers 订阅源）
├── 规则集（Rule Providers）
└── 规则（路由规则链）
```

## 5.2 全局配置

| 参数 | 值 | 说明 |
|------|-----|------|
| `mixed-port` | 7890 | 混合代理端口（HTTP + SOCKS5） |
| `socks-port` | 7891 | SOCKS5 代理端口 |
| `allow-lan` | true | 允许局域网访问 |
| `mode` | rule | 规则路由模式 |
| `external-controller` | 127.0.0.1:9090 | WebUI API 地址 |
| `external-ui` | ./ui | WebUI 静态文件路径 |
| `log-level` | info | 日志级别 |
| `ipv6` | true | 启用 IPv6 |
| `tcp-concurrent` | true | TCP 并发 |
| `unified-delay` | true | 统一延迟显示 |

## 5.3 TUN 配置

TUN 模式创建一个虚拟网卡，所有系统流量都会经过 mihomo：

```yaml
tun:
  enable: true          # 启用 TUN
  device: Meta          # 虚拟网卡名称
  stack: mixed          # 网络栈：system/gvisor/mixed
  dns-hijack:           # 劫持 DNS 查询
    - any:53
  auto-route: true      # 自动设置全局路由
  auto-detect-interface: true  # 自动识别出站网卡
```

## 5.4 DNS 配置（Fake-IP 模式）

Fake-IP 模式为每个域名分配一个假的 IP 地址（198.18.0.0/16 段），避免真实 DNS 查询延迟。mihomo 会拦截对这些假 IP 的访问并匹配规则。

```
DNS 查询流程：

用户请求 example.com
      ↓
mihomo 返回 fake-ip（198.18.x.x）
      ↓
用户发起对 198.18.x.x 的连接
      ↓
mihomo 根据域名规则选择代理/直连
      ↓
代理节点发起真实 DNS 查询 → 建立连接
```

## 5.5 代理组

| 代理组 | 类型 | 说明 |
|--------|------|------|
| `Auto` | url-test | 自动选择延迟最低的节点 |
| `PROXY` | select | 手动选择代理节点 |
| `OpenAI` | select | ChatGPT / OpenAI API |
| `Claude` | select | Claude AI |
| `GitHub` | select | GitHub 系列服务 |
| `Netflix` | select | Netflix 流媒体 |
| `Disney` | select | Disney+ 流媒体 |
| `Youtube` | select | YouTube |
| `Spotify` | select | Spotify |
| `Tiktok` | select | TikTok |
| `Telegram` | select | Telegram |
| `Twitter` | select | Twitter/X |
| `Copilot` | select | Microsoft Copilot / Bing AI |
| `OneDrive` | select | Microsoft OneDrive |

## 5.6 路由规则链

规则按顺序匹配，命中即终止：

```
1. 应用程序直连（RULE-SET applications → DIRECT）
2. 内网/私有地址（RULE-SET private → DIRECT）
3. iCloud/Apple 服务（DIRECT）
4. 中国 CDN/游戏（GEOSITE cn → DIRECT）
5. 特定域名直连（mirrors.tuna, chat.qwen.ai 等 → DIRECT）
6. Claude / GitHub / Copilot → 对应代理组
7. OpenAI / Netflix / YouTube / Telegram 等 → 对应代理组
8. GFW 列表 → PROXY
9. 国内 IP → DIRECT
10. 其余流量 → PROXY（白名单模式）
```

# 6. 自定义配置

## 6.1 配置变体

`my_config/` 目录下提供了多个配置变体：

| 文件 | 说明 |
|------|------|
| `config_bak.yaml` | 完整带注释的配置，含订阅链接（已脱敏） |
| `config01.yaml` | 配置变体 1，不含 direct-nameserver 等参数 |
| `config02.yaml` | 配置变体 2，含 direct-nameserver 和 proxy-server-nameserver |

## 6.2 自定义规则

在 `rules:` 部分添加新规则：

```yaml
rules:
  # 格式：规则类型,匹配条件,策略组
  - DOMAIN-SUFFIX,example.com,PROXY    # 域名后缀匹配
  - DOMAIN-KEYWORD,example,PROXY       # 域名关键词匹配
  - DOMAIN,example.com,PROXY            # 精确域名匹配
  - GEOSITE,youtube,Youtube             # GeoSite 类别匹配
  - GEOIP,CN,DIRECT                     # IP 地理位置匹配
  - MATCH,PROXY                         # 兜底规则
```

## 6.3 添加订阅源

在 `proxy-providers:` 部分添加新的机场订阅：

```yaml
proxy-providers:
  my_provider:                          # 自定义名称
    type: http
    url: "你的订阅链接"
    path: ./proxy_providers/my_provider.yaml
    interval: 86400                     # 自动更新间隔（秒）
    health-check:
      enable: true
      interval: 600
      timeout: 5000
      url: https://www.gstatic.com/generate_204
```

然后在 `proxy-groups` 中引用 `use`：

```yaml
proxy-groups:
  - name: "PROXY"
    type: select
    use:
      - my_provider
```

# 7. 故障排除

## 7.1 防火墙设置 (Firewalld/UFW)

如果你使用了防火墙（如 `firewalld`），需要手动放行 TUN 网卡：

> 下面提到的 Meta 是你开启 TUN 模式创建的虚拟网卡，通过指令 `ip a` 查看。

- **Firewalld**:

```bash
sudo firewall-cmd --permanent --zone=trusted --add-interface=Meta
sudo firewall-cmd --reload
```

- **UFW**:

```bash
sudo ufw allow in on Meta
```

## 7.2 配置 Systemd 服务权限

### 方式一（推荐）

如果通过 `pacman` 安装，通常自带了 service 文件，你可以使用 systemd 自带的交互式命令创建覆盖配置：

```bash
sudo systemctl edit mihomo
```

在打开的编辑器中，输入以下内容：

```ini
[Service]
# 如果你想以 root 运行以避开所有权限烦恼，取消下面两行的注释
# User=root
# Group=root

# 为 TUN 模式和 53 端口添加权限（针对非 root 用户）
AmbientCapabilities=CAP_NET_BIND_SERVICE CAP_NET_ADMIN
CapabilityBoundingSet=CAP_NET_BIND_SERVICE CAP_NET_ADMIN

# 如果你想指定特定的配置文件路径
# ExecStart=
# ExecStart=/usr/bin/mihomo -d /etc/mihomo
```

> **注意**：如果你要覆盖 `ExecStart` 这种带有命令的参数，必须先写一行空的 `ExecStart=` 来清除旧指令，然后再写一行新的。

保存退出编辑器并重启服务：

```bash
sudo systemctl restart mihomo
```

查看最终合并后的配置：

```bash
systemctl cat mihomo
```

### 方式二

编辑（或创建）服务文件 `/etc/systemd/system/mihomo.service`：

```ini
[Unit]
Description=Mihomo Daemon
After=network.target

[Service]
Type=simple
User=your-user-name
Group=your-group-name
ExecStart=/usr/bin/mihomo -d /etc/mihomo
Restart=always

AmbientCapabilities=CAP_NET_BIND_SERVICE CAP_NET_ADMIN
CapabilityBoundingSet=CAP_NET_BIND_SERVICE CAP_NET_ADMIN

[Install]
WantedBy=multi-user.target
```

> 不推荐该方式，更新 mihomo 后可能会导致配置失效。

## 7.3 开启内核转发

创建文件 `/etc/sysctl.d/99-ip-forward.conf`：

```conf
net.ipv4.ip_forward = 1
# 如果需要 IPv6
net.ipv6.conf.all.forwarding = 1
```

然后执行 `sudo sysctl --system` 生效。

## 7.4 路由设置 (nftables/iptables)

创建文件 `/etc/sysctl.d/99-rp-filter.conf`：

```conf
net.ipv4.conf.all.rp_filter = 2
net.ipv4.conf.default.rp_filter = 2
# 针对 config.yaml 中设置的 tun 接口（Meta）
net.ipv4.conf.Meta.rp_filter = 2
```

## 7.5 检查配置是否正确

```bash
sudo mihomo -t -d /etc/mihomo
```

如果输出 `configuration file test passed`，则配置无误。

## 7.6 常见问题

**Q: WebUI 无法访问**

检查 `external-controller` 和 `external-ui` 配置，确认 UI 文件路径正确。

**Q: WebUI 访问不一致，比如用的 zashboard，在浏览器中输入 `127.0.0.1:9090/ui` 显示的却是 metacubexd**

请打开你的 config.yaml 配置文件，找到 external-ui 相关的控制块（通常在配置文件的最顶部几行），按照以下规范进行修改：

```yaml
# 1. 指定本地存放 WebUI 面板的文件夹名称（你可以自定义，例如叫 ui 或者是 zashboard）
external-ui: ui

# 2. 【核心】将下载地址显式指定为 zashboard 的最新打包发布地址
# 推荐使用 dist-cdn-fonts.zip，这个版本去除了本地大字体文件，加载和下载速度极快
external-ui-url: "https://github.com/Zephyruso/zashboard/releases/latest/download/dist-cdn-fonts.zip"
```

修改完成后的操作：

1. 保存配置文件。
2. 彻底关闭并重启你的 mihomo 裸核（如果是在 Linux/OpenWrt 上，使用 systemctl restart mihomo 或者是软路由后台重启插件）。
3. 关键一步：由于浏览器会深度缓存前端样式，请在浏览器中按下 Ctrl + Shift + Delete，清除该端口（如 127.0.0.1:9090）的所有缓存和站点数据。
4. 换个无痕模式（隐身窗口）再次打开，看看是否恢复成了 Zashboard 那标志性的 Apple 风格/极简 UI。

**Q: TUN 网卡没有创建**

检查 systemd 权限配置，确保有 `CAP_NET_ADMIN`。查看日志：`sudo journalctl -fu mihomo.service`。

**Q: DNS 解析异常**

检查 `default-nameserver` 是否配置了可用的 DNS 服务器。Fake-IP 模式下 `default-nameserver` 必须使用纯 IP 而非 DoH/DoT。

**Q: 代理节点连接失败**

确认订阅链接是否有效，执行 `sudo mihomo -d /etc/mihomo` 前台运行查看具体错误日志。

---

## 许可证

[MIT](LICENSE) © 2025 loskyertt
