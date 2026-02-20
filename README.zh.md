# UFW-Cloudflare

🌐 [English](README.md) | [Português](README.pt-BR.md) | [Español](README.es.md) | **中文** | [Íslenska](README.is.md)

自动管理 UFW（Uncomplicated Firewall）中的 [Cloudflare IP 范围](https://www.cloudflare.com/ips/)。此脚本获取最新的 Cloudflare IPv4 和 IPv6 地址，并为 HTTP/HTTPS 流量（端口 80 和 443 TCP）创建防火墙允许规则，确保只有通过 Cloudflare 代理的请求才能到达您的源服务器。

专为 **Ubuntu 服务器**设计（也兼容基于 Debian 的系统）。

## 为什么需要？

当您的域名通过 Cloudflare 代理时，所有访客流量都来自 [Cloudflare IP 地址](https://developers.cloudflare.com/fundamentals/concepts/cloudflare-ip-addresses/)，而不是访客的个人 IP。您的防火墙必须允许这些范围，否则合法流量将被阻止。此脚本自动化了这一过程并保持规则更新。

## 要求

- **Ubuntu 20.04+**（或基于 Debian 的系统）
- 已安装并启用 `ufw`
- `wget`
- Root 权限（`sudo`）

## 快速开始

```bash
wget -O ufw.sh https://raw.githubusercontent.com/hyzis-manager/ufw-cloudflare/main/ufw.sh
chmod +x ufw.sh
sudo ./ufw.sh
```

首次运行时，脚本将：

1. 从 `cloudflare.com/ips-v4` 和 `cloudflare.com/ips-v6` 获取当前的 Cloudflare IPv4 和 IPv6 范围
2. 为每个范围在端口 **80** 和 **443**（TCP）上创建 UFW 允许规则
3. 询问您是否要启用 **Supervision**（每日自动更新）

## 用法

```
sudo ./ufw.sh [选项]
```

| 选项 | 缩写 | 描述 |
|---|---|---|
| `--help` | `-h` | 显示帮助信息 |
| `--purge` | `-p` | 删除所有现有的 Cloudflare 规则（标记为 `#cloudflare` 注释的规则），然后添加新规则 |
| `--no-new` | `-n` | 不获取 IP，不添加新规则（与 `--purge` 一起使用可仅删除规则） |
| `--supervision` | `-s` | 通过 systemd 定时器启用每日自动更新 |

## 示例

**首次设置** — 获取并允许所有 Cloudflare IP：

```bash
sudo ./ufw.sh
```

**刷新规则** — 删除旧规则并添加当前规则：

```bash
sudo ./ufw.sh --purge
```

**删除所有 Cloudflare 规则**而不添加新规则：

```bash
sudo ./ufw.sh --purge --no-new
```

**启用每日自动更新**（Supervision）：

```bash
sudo ./ufw.sh --supervision
```

## Supervision

Supervision 功能安装一个 **systemd 定时器**，每 24 小时使用 `--purge` 运行一次脚本，确保您的防火墙规则始终反映最新的 Cloudflare IP 范围。

启用后，将创建两个 systemd 单元：

- `ufw-cloudflare-supervision.service` — 执行脚本的 oneshot 服务
- `ufw-cloudflare-supervision.timer` — 每天触发服务的定时器（启动后 5 分钟也会触发）

### 管理定时器

```bash
# 检查定时器状态
systemctl status ufw-cloudflare-supervision.timer

# 查看下次计划运行时间
systemctl list-timers ufw-cloudflare-supervision.timer

# 禁用自动更新
sudo systemctl stop ufw-cloudflare-supervision.timer
sudo systemctl disable ufw-cloudflare-supervision.timer

# 手动触发更新
sudo systemctl start ufw-cloudflare-supervision.service
```

## 工作原理

1. 从 `https://www.cloudflare.com/ips-v4` 获取 IPv4 范围
2. 从 `https://www.cloudflare.com/ips-v6` 获取 IPv6 范围
3. 对每个 CIDR 范围执行：`ufw allow from <IP> to any port 80,443 proto tcp comment "cloudflare"`
4. 使用 `--purge` 时，在添加新规则之前删除所有标记为 `# cloudflare` 注释的 UFW 规则
5. 在应用更改之前验证 IP 格式并确认下载成功

## 输出图例

执行过程中，脚本显示进度指示器：

- **`+`**（绿色）— 规则已创建
- **`-`**（红色）— 规则已删除
- **`.`**（灰色）— 规则已跳过（已存在或无效）

## Cloudflare IP 范围

脚本直接从 Cloudflare 官方端点获取 IP。这些范围也可通过 [Cloudflare API](https://developers.cloudflare.com/api/resources/ips/methods/list/) 在 `https://api.cloudflare.com/client/v4/ips` 获取（无需认证）。Cloudflare 不经常更新这些范围，但建议保持您的允许列表为最新状态。

## 许可证

MIT
