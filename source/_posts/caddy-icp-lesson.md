---
title: Caddy + Cloudflare + 阿里云 ICP 备案踩坑记录
date: 2026-06-09 15:20:00
tags:
  - 运维
  - Caddy
  - Cloudflare
---

## 背景

手里有一台阿里云 ECS（Ubuntu 22.04），IP `SERVER_IP`，上面跑着一个服务监听 `12201` 端口。

想通过域名 `f.example.com` 用 HTTPS 访问这个服务，于是开始了折腾之旅。

## 第一步：安装 Caddy

Caddy 是现代 Web 服务器，自动 HTTPS 是卖点。SSH 到服务器后：

```bash
apt update
apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list
apt update && apt install caddy -y
```

配置 `/etc/caddy/Caddyfile`：

```
f.example.com {
    reverse_proxy localhost:12201
}
```

重启 Caddy：`systemctl restart caddy`

## 第二步：DNS 配置

域名 `example.com` 托管在 Cloudflare，添加 A 记录：

- 类型: A
- 名称: f
- 内容: `SERVER_IP`
- 代理: 橙色云朵（开启 Cloudflare 代理）

## 第三步：SSL 525 错误

访问 `https://f.example.com`，返回 **Cloudflare 525 SSL Handshake Failed**。

原因分析：

- Caddy 尝试用 Let's Encrypt 申请证书
- 但 Cloudflare 代理拦截了 HTTP-01 验证请求
- 证书申请失败，Caddy 没有有效证书
- Cloudflare 用 HTTPS 连源站，握手失败

### 解决方案：Cloudflare Origin Certificate

在 Cloudflare 面板：`SSL/TLS` → `源服务器` → `创建证书`

获得证书和私钥后，上传到服务器：

```bash
mkdir -p /etc/caddy/certs
# 上传 cert.pem 和 key.pem 到 /etc/caddy/certs/
chown -R caddy:caddy /etc/caddy/certs
```

修改 Caddyfile：

```
f.example.com {
    tls /etc/caddy/certs/cert.pem /etc/caddy/certs/key.pem
    reverse_proxy localhost:12201
}
```

重启后 SSL 525 错误解决。

## 第四步：ICP 备案拦截

以为大功告成，结果访问返回 **403 Forbidden**，页面内容：

```html
<title>Non-compliance ICP Filing</title>
```

阿里云会对未备案域名拦截 80/443 端口的 HTTP/HTTPS 请求。这是中国法规要求，云服务商强制执行。

### 尝试绕过

关闭 Cloudflare 代理（灰色云朵），让流量直连服务器 → 依然被拦截。

开启 Cloudflare 代理（橙色云朵）→ 流量经过 Cloudflare 到源站，源站依然返回 ICP 拦截页。

结论：**只要源站在国内，80/443 端口的请求都会被阿里云拦截**，无论是否经过 Cloudflare。

## 最终方案

使用非标端口（如 8443）绕过 ICP 检查：

```
:8443 {
    tls /etc/caddy/certs/cert.pem /etc/caddy/certs/key.pem
    reverse_proxy localhost:12201
}
```

访问地址变为 `https://f.example.com:8443`。

注意：阿里云安全组需要放行 8443 端口（TCP 入方向）。

## 踩坑总结

1. **Caddy 自动 HTTPS + Cloudflare 代理冲突**：Cloudflare 拦截了 Let's Encrypt 的验证请求，需要手动配置 Cloudflare Origin Certificate
2. **阿里云 ICP 备案拦截**：国内服务器 80/443 端口对未备案域名强制拦截，无法通过 Cloudflare 绕过
3. **Cloudflare Origin Certificate 私钥格式**：粘贴时注意行尾空格，可能导致 SSL 握手失败
4. **fail2ban 封禁**：频繁 SSH 连接可能触发 fail2ban，导致无法远程管理服务器

## 经验教训

- 国内服务器 + 未备案域名 = 只能用非标端口
- Cloudflare 代理 + 自定义证书 = 需要 Origin Certificate
- 域名解析用 Cloudflare DNS，但代理功能根据需求开关
- 重要操作前先检查 fail2ban 状态，避免被封 IP
