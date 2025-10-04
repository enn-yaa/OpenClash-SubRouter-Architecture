# 🧭 OpenClash-SubRouter-Architecture

> OpenClash 二级软路由架构研究与 DNS 优化方案  
> A practical DNS architecture for OpenClash running under sub-router environments (ROS + OpenWrt)

---

## 📘 项目简介

本项目致力于构建一套适用于 **二级软路由环境（主路由拨号 + 下级软路由代理）** 的 OpenClash DNS 配置与分层网络架构方案。

不同于传统的 `Custom_OpenClash_Rules` 等“规则级”优化，本项目专注于：

- **DNS 分层架构设计**（DoH / Fake-IP / GeoIP 策略）  
- **ROS ↔ OpenWrt 协同防火墙与 NAT 配置**  
- **家庭网络分层模型（主路由 + 二级软路由 + IoT/NAS）**  
- **面向工程师的自动化部署与实测优化**

---

## 🧩 架构总览

```
[Internet]
   │ (PPPoE 拨号)
[主路由] MikroTik ROS5009
   │
   └── [WAN:192.168.88.2] [R4S - OpenWrt + OpenClash]
         └── [LAN:10.0.0.1] → 终端设备 (PC / TV / iPad)
```

**架构特点：**
- 主路由专注拨号与 NAT  
- 二级路由负责策略分流与 DNS 控制  
- 分层网络互不干扰，维护简单，稳定高效。

---

## ⚙️ 推荐 OpenClash DNS 配置

```yaml
dns:
  enable: true
  ipv6: false
  listen: :53
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16

  default-nameserver:
    - 223.5.5.5
    - 119.29.29.29

  nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query

  fallback:
    - https://1.1.1.1/dns-query
    - https://dns.google/dns-query
    - https://cloudflare-dns.com/dns-query

  fallback-filter:
    geoip: true
    ipcidr:
      - 240.0.0.0/4
```

### 🧠 配合 dnsmasq 设置

- 忽略系统解析文件 `/etc/resolv.conf`
- 不转发 DNS 查询到上游服务器
- “DNS 转发” 填写：`127.0.0.1#53`

### 🔒 ROS 放行规则

```bash
/ip firewall nat
add chain=dstnat action=redirect to-ports=53 protocol=udp dst-port=53 src-address=!192.168.88.2 comment="Allow R4S DNS bypass"
```

---

## 🚀 架构优势

| 特性 | 说明 |
|------|------|
| 分层设计 | 主路由拨号、二级软路由分流 |
| DNS 独立 | 无 SmartDNS 依赖，避免循环冲突 |
| 高兼容性 | 支持 Fake-IP + GeoIP |
| 稳定安全 | ROS 防火墙防止运营商劫持 |
| 可扩展 | 可并行部署 NAS、Docker、AdGuard 等服务 |

---

## 🧠 面向工程师的讨论话题

- 二级路由下的 DNS 分流逻辑是否可以更智能？  
- 是否可以实现 ROS ↔ OpenWrt 自动 NAT 配置脚本？  
- Fake-IP 与 Redir-Host 模式能否动态切换？  
- 如何在双 NAT 架构下实现 IPv6 分流？  
- Docker/容器化环境下的 Clash 网络隔离优化？

---

## 💬 开源协作邀请

目前国内外对「二级软路由环境下 OpenClash 的 DNS 分层架构」研究仍非常有限，
希望更多开发者、网络工程师、家庭网络爱好者参与：

- 分享你的实测配置与延迟数据；  
- 提交优化脚本或防火墙模板；  
- 共同完善适配文档与自动化工具；  
- 翻译、传播、让更多人理解分层网络理念。

📬 欢迎提交 PR / Issues / 博客链接，  
让我们共同构建下一代家庭网络标准化方案。

---

## 🧭 作者与贡献

- 主作者：enn-yaa 
- 测试设备：R4S、ROS5009、OpenWrt 23.05.6  
- 协议：MIT License  
- 项目主页：[https://github.com/yourname/OpenClash-SubRouter-Architecture](https://github.com/enn-yaa/OpenClash-SubRouter-Architecture)

> _“让路由不止是通网的盒子，而是连接世界的系统。”_
