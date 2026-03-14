# 🔒 OpenClaw 云端安全保护方案

> **版本：** v1.0  
> **日期：** 2026-03-09  
> **适用：** 阿里云 ECS 部署的 OpenClaw Gateway  
> **目标：** 防止公网暴露，确保 API 安全

---

## 📋 目录

1. [安全风险分析](#安全风险分析)
2. [阿里云安全组配置](#阿里云安全组配置)
3. [Nginx 反向代理认证](#nginx-反向代理认证)
4. [API 接口安全防护](#api-接口安全防护)
5. [防止公网暴露方案](#防止公网暴露方案)
6. [OpenClaw 安全部署清单](#openclaw-安全部署清单)
7. [快速加固脚本](#快速加固脚本)
8. [参考链接](#参考链接)

---

## 安全风险分析

### 🚨 当前风险（公网直接暴露）

| 风险 | 严重性 | 描述 |
|------|--------|------|
| **未授权访问** | 🔴 高危 | 任何人可直接调用 API |
| **API 滥用** | 🔴 高危 | 无速率限制，易被 DDoS |
| **数据泄露** | 🔴 高危 | 敏感数据明文传输 |
| **暴力破解** | 🟡 中危 | 无 fail2ban 防护 |
| **中间人攻击** | 🟡 中危 | 未强制 HTTPS |

### 🎯 加固目标

- ✅ 仅允许授权用户访问
- ✅ 所有流量加密（HTTPS）
- ✅ 防止暴力破解和 DDoS
- ✅ 最小化攻击面
- ✅ 完整的审计日志

---

## 阿里云安全组配置

### 1.1 核心原则

**默认拒绝所有入站，仅开放必要端口**

### 1.2 推荐配置

| 规则 | 端口 | 源 IP | 说明 |
|------|------|-------|------|
| SSH | 22 | 管理 IP/32 | 仅允许你的 IP |
| HTTPS | 443 | 0.0.0.0/0 | 公开 Web 服务 |
| HTTP | 80 | 0.0.0.0/0 | 重定向到 HTTPS |
| **其他** | **全部** | **拒绝** | **默认拒绝** |

### 1.3 配置步骤（图文指南）

**步骤 1：登录阿里云控制台**

1. 访问 https://ecs.console.aliyun.com/
2. 使用账号密码登录
3. 左侧菜单 → **网络与安全** → **安全组**

📎 **图文教程：** https://help.aliyun.com/document_detail/25471.html

**步骤 2：找到你的安全组**

```
1. 在安全组列表中找到你的 ECS 实例关联的安全组
2. 点击安全组 ID 进入详情
3. 点击"入方向"标签页
```

**步骤 3：添加入站规则**

```
1. 点击"手动添加"按钮
2. 填写以下信息：
   - 规则方向：入方向
   - 策略：允许
   - 优先级：1
   - 协议类型：TCP
   - 端口范围：22/22
   - 授权对象：你的管理 IP/32（如 123.45.67.89/32）
   - 描述：SSH 管理（仅信任 IP）
3. 点击"保存"
```

📸 **截图参考：** 阿里云控制台 → 安全组 → 入方向规则列表

**步骤 4：配置 HTTPS 规则**

```
1. 再次点击"手动添加"
2. 填写：
   - 规则方向：入方向
   - 策略：允许
   - 优先级：2
   - 协议类型：TCP
   - 端口范围：443/443
   - 授权对象：0.0.0.0/0（公开访问）
   - 描述：HTTPS Web 服务
3. 点击"保存"
```

**步骤 5：配置 HTTP 规则（用于重定向）**

```
1. 点击"手动添加"
2. 填写：
   - 规则方向：入方向
   - 策略：允许
   - 优先级：3
   - 协议类型：TCP
   - 端口范围：80/80
   - 授权对象：0.0.0.0/0
   - 描述：HTTP 重定向
3. 点击"保存"
```

**步骤 6：删除危险规则**

```
检查并删除以下规则：
- ❌ 允许 0.0.0.0/0 访问 22 端口
- ❌ 允许所有端口（0/0）
- ❌ 任何不必要的开放规则

删除方法：
1. 找到要删除的规则
2. 点击右侧"删除"按钮
3. 确认删除
```

📎 **官方图文教程：** https://help.aliyun.com/document_detail/25471.html

### 1.4 快速检查

```bash
# 查看当前开放端口
sudo netstat -tlnp

# 应该只看到：
# - 127.0.0.1:8080 (OpenClaw Gateway)
# - 0.0.0.0:443 (Nginx HTTPS)
# - 0.0.0.0:80 (Nginx HTTP)
```

---

## Nginx 反向代理认证

### 2.1 方案选择

| 方案 | 安全性 | 复杂度 | 推荐场景 |
|------|--------|--------|----------|
| HTTP Basic Auth | ⭐⭐⭐ | 简单 | 个人使用 |
| JWT Token | ⭐⭐⭐⭐ | 中等 | API 访问 |
| OAuth2 | ⭐⭐⭐⭐⭐ | 复杂 | 企业级 |

### 2.2 HTTP Basic Auth 配置（推荐个人使用）

📎 **图文教程：** https://www.digitalocean.com/community/tutorials/how-to-set-up-password-authentication-with-nginx

**步骤 1：安装工具**
```bash
sudo apt update
sudo apt install apache2-utils nginx -y
```

**步骤 2：创建密码文件**
```bash
# 创建目录
sudo mkdir -p /etc/nginx/auth

# 创建密码文件（会提示输入密码，输入两次）
sudo htpasswd -c /etc/nginx/auth/.htpasswd openclaw

# 添加更多用户（可选，不使用 -c）
sudo htpasswd /etc/nginx/auth/.htpasswd anotheruser

# 查看密码文件内容（可选）
cat /etc/nginx/auth/.htpasswd
```

**步骤 3：配置 Nginx**

```bash
# 编辑 Nginx 配置
sudo nano /etc/nginx/sites-available/openclaw
```

```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    # 强制 HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    # SSL 证书（使用 Let's Encrypt）
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # 安全响应头
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # 访问日志
    access_log /var/log/nginx/openclaw_access.log;
    error_log /var/log/nginx/openclaw_error.log;

    location / {
        # 启用认证
        auth_basic "OpenClaw Gateway";
        auth_basic_user_file /etc/nginx/auth/.htpasswd;

        # 反向代理到 OpenClaw
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 超时设置
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # 公开的健康检查端点
    location /health {
        access_log off;
        auth_basic off;
        return 200 "OK\n";
        add_header Content-Type text/plain;
    }
}
```

**步骤 4：启用配置**
```bash
# 创建软链接
sudo ln -s /etc/nginx/sites-available/openclaw /etc/nginx/sites-enabled/

# 测试配置
sudo nginx -t

# 重载 Nginx
sudo systemctl reload nginx
```

### 2.3 访问测试

```bash
# 无认证访问（应该返回 401）
curl https://your-domain.com/api/status

# 带认证访问（应该正常）
curl -u openclaw https://your-domain.com/api/status
```

---

## API 接口安全防护

### 3.1 速率限制配置

**防止 API 滥用和 DDoS 攻击**

```nginx
http {
    # 定义限流区域（每个 IP 每秒 10 个请求）
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
    
    server {
        location /api/ {
            # 应用限流（允许突发 20 个请求）
            limit_req zone=api_limit burst=20 nodelay;
            limit_req_status 429;
            
            # 响应头
            add_header X-RateLimit-Limit 10;
            add_header X-RateLimit-Remaining $limit_req_remaining;
            
            proxy_pass http://127.0.0.1:8080;
        }
    }
}
```

### 3.2 Fail2ban 防暴力破解

**步骤 1：安装**
```bash
sudo apt install fail2ban -y
```

**步骤 2：创建 Nginx 认证保护配置**
```bash
sudo nano /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
bantime = 3600  # 封禁 1 小时
findtime = 600  # 10 分钟内
maxretry = 5    # 最多 5 次失败

[nginx-auth]
enabled = true
port = http,https
filter = nginx-auth
logpath = /var/log/nginx/*error.log
```

**步骤 3：创建过滤器**
```bash
sudo nano /etc/fail2ban/filter.d/nginx-auth.conf
```

```ini
[Definition]
failregex = ^.*user ".*": authentication failure for ".*": password mismatch.*$
ignoreregex =
```

**步骤 4：启动 Fail2ban**
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# 查看状态
sudo fail2ban-client status nginx-auth
```

### 3.3 OWASP API 安全建议

| 风险 | 防护措施 |
|------|----------|
| 认证失效 | 使用 JWT/OAuth2，令牌轮换 |
| 速率限制 | 每 IP 限流，配额管理 |
| 敏感数据 | HTTPS 加密，字段白名单 |
| 注入攻击 | 输入验证，参数化查询 |

---

## 防止公网暴露方案

### 4.1 方案对比

| 方案 | 安全性 | 便利性 | 推荐度 |
|------|--------|--------|--------|
| 安全组 IP 白名单 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| SSH 隧道 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| WireGuard VPN | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Tailscale | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

### 4.2 方案 A：SSH 隧道（推荐）

**无需暴露任何端口，通过 SSH 访问**

```bash
# 本地端口转发
ssh -L 8080:localhost:8080 user@your-server-ip

# 然后在本地访问
curl http://localhost:8080/api/status
```

**永久隧道配置（~/.ssh/config）**
```
Host openclaw
    HostName your-server-ip
    User openclaw
    LocalForward 8080 localhost:8080
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

**使用：**
```bash
ssh openclaw
```

### 4.3 方案 B：Tailscale（最简单）

**自动组网，无需公网 IP**

**步骤 1：安装 Tailscale**
```bash
# 服务器端
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# 本地电脑
# macOS: brew install tailscale
# Windows: 下载安装包
# Linux: 同上
```

**步骤 2：认证**
```bash
# 按提示访问链接并登录
```

**步骤 3：使用 Tailscale IP 访问**
```bash
# 查看服务器的 Tailscale IP
tailscale ip

# 直接访问（无需暴露公网端口）
curl http://100.x.y.z:8080/api/status
```

### 4.4 方案 C：UFW 防火墙

**Ubuntu 系统防火墙配置**

```bash
# 1. 启用 IPv6
sudo nano /etc/default/ufw
# 设置 IPV6=yes

# 2. 设置默认策略
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 3. 允许 SSH（先配置再启用！）
sudo ufw allow from 你的管理 IP to any port 22 proto tcp

# 4. 允许 Nginx
sudo ufw allow 'Nginx Full'

# 5. 启用防火墙
sudo ufw enable

# 6. 查看状态
sudo ufw status verbose
```

---

## OpenClaw 安全部署清单

### ✅ 系统层面

- [ ] 创建专用用户（非 root）
- [ ] 配置 SSH 密钥登录
- [ ] 禁用 SSH 密码认证
- [ ] 修改 SSH 端口（非 22）
- [ ] 安装 fail2ban
- [ ] 配置 UFW 防火墙
- [ ] 定期系统更新

### ✅ 网络层面

- [ ] 安全组仅开放 80/443
- [ ] SSH 限制源 IP
- [ ] 删除默认允许规则
- [ ] 启用安全组日志

### ✅ 应用层面

- [ ] Nginx 反向代理
- [ ] 启用 HTTPS（Let's Encrypt）
- [ ] 配置 HTTP Basic Auth
- [ ] 设置速率限制
- [ ] 配置安全响应头
- [ ] 启用访问日志

### ✅ 监控层面

- [ ] 配置日志轮转
- [ ] 设置监控告警
- [ ] 定期备份配置
- [ ] 定期安全审计

---

## 快速加固脚本

### 一键安全检查脚本

```bash
#!/bin/bash
# openclaw-security-check.sh

echo "🔒 OpenClaw 安全检查"
echo "=================="

# 检查防火墙
echo -e "\n📊 防火墙状态:"
sudo ufw status verbose

# 检查开放端口
echo -e "\n📊 开放端口:"
sudo netstat -tlnp | grep LISTEN

# 检查 SSH 配置
echo -e "\n📊 SSH 配置:"
grep -E "^(PermitRootLogin|PasswordAuthentication|PubkeyAuthentication)" /etc/ssh/sshd_config

# 检查 Nginx 状态
echo -e "\n📊 Nginx 状态:"
sudo systemctl status nginx --no-pager

# 检查 fail2ban
echo -e "\n📊 Fail2ban 状态:"
sudo fail2ban-client status 2>/dev/null || echo "未安装 fail2ban"

# 检查 SSL 证书
echo -e "\n📊 SSL 证书:"
sudo ls -la /etc/letsencrypt/live/*/ 2>/dev/null || echo "未配置 Let's Encrypt"

echo -e "\n✅ 检查完成"
```

**使用方法：**
```bash
# 下载脚本
curl -O https://raw.githubusercontent.com/XueLuo9527/meory-place/main/openclaw-security-check.sh

# 赋予执行权限
chmod +x openclaw-security-check.sh

# 运行
sudo ./openclaw-security-check.sh
```

### 一键加固脚本（谨慎使用）

```bash
#!/bin/bash
# openclaw-hardening.sh

set -e

echo "🔒 OpenClaw 安全加固脚本"
echo "======================"
echo "⚠️ 警告：此脚本会修改系统配置，请确保你了解每一步操作"
read -p "按回车继续，Ctrl+C 取消..."

# 1. 创建专用用户
echo -e "\n📝 创建 openclaw 用户..."
if ! id "openclaw" &>/dev/null; then
    sudo useradd -m -s /bin/bash openclaw
    echo "✅ 用户创建成功"
else
    echo "⚠️  用户已存在"
fi

# 2. 安装必要软件
echo -e "\n📦 安装软件..."
sudo apt update
sudo apt install -y nginx fail2ban ufw

# 3. 配置 UFW
echo -e "\n🔥 配置 UFW 防火墙..."
sudo ufw default deny incoming
sudo ufw default allow outgoing
echo "⚠️  请添加你的管理 IP 到白名单："
read -p "你的管理 IP: " ADMIN_IP
sudo ufw allow from $ADMIN_IP to any port 22 proto tcp
sudo ufw allow 'Nginx Full'
sudo ufw --force enable

# 4. 配置 Nginx
echo -e "\n🌐 配置 Nginx..."
sudo mkdir -p /etc/nginx/auth
sudo htpasswd -c /etc/nginx/auth/.htpasswd openclaw

# 5. 配置 fail2ban
echo -e "\n👮 配置 fail2ban..."
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

echo -e "\n✅ 加固完成！"
echo -e "\n📋 下一步："
echo "1. 配置 Nginx 反向代理（参考上文）"
echo "2. 申请 SSL 证书：sudo certbot --nginx"
echo "3. 测试访问：curl -u openclaw https://your-domain.com"
```

---

## 参考链接

### 阿里云官方（图文教程）

| 主题 | 链接 | 类型 |
|------|------|------|
| 安全组配置 | https://help.aliyun.com/zh/ecs/user-guide/start-using-security-groups | 官方图文 |
| ECS 防火墙 | https://help.aliyun.com/zh/ecs/user-guide/security-group-overview | 官方图文 |
| SSH 密钥对 | https://help.aliyun.com/zh/ecs/user-guide/configure-the-ssh-key-pair | 官方图文 |
| 云盾安骑士 | https://help.aliyun.com/product/28306.html | 官方图文 |
| SSL 证书申请 | https://help.aliyun.com/document_detail/35191.html | 官方图文 |

### Nginx 配置（中文图文教程）

| 主题 | 链接 | 来源 |
|------|------|------|
| Nginx HTTP Basic Auth 配置 | https://cloud.tencent.com/developer/article/1157921 | 腾讯云 |
| Nginx 用户认证配置 | https://www.ttlsa.com/nginx/nginx-basic-http-auth/ | TTLSA |
| Nginx 反向代理配置 | https://segmentfault.com/a/1190000047615317 | SegmentFault |
| Let's Encrypt 配置 | https://certbot.eff.org/instructions | 官方交互指南 |
| Nginx 限流配置 | https://blog.csdn.net/weixin_43114209/article/details/108683356 | CSDN |

### 安全工具（中文教程）

| 工具 | 链接 | 来源 |
|------|------|------|
| fail2ban 防暴力破解 | https://blog.csdn.net/java_zdc/article/details/108683356 | CSDN |
| UFW 防火墙配置 | https://www.cnblogs.com/Cocolone/articles/13789012.html | 博客园 |
| Tailscale 组网 | https://zhuanlan.zhihu.com/p/71112431 | 知乎 |
| WireGuard VPN | https://www.wireguard.com/zh_CN/quickstart/ | 官方中文 |

### OWASP 安全标准

| 主题 | 链接 | 类型 |
|------|------|------|
| API Security Top 10 | https://owasp.org/www-project-api-security/ | 官方文档 |
| REST 安全清单 | https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html | 清单 |
| 认证清单 | https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html | 清单 |

### 视频教程

| 主题 | 平台 | 搜索关键词 |
|------|------|------------|
| 阿里云安全组配置 | B 站 | 阿里云安全组教程 |
| Nginx 反向代理 | B 站 | Nginx 反向代理教程 |
| Let's Encrypt 免费证书 | B 站 | Let's Encrypt 配置 |
| fail2ban 防暴力破解 | YouTube | fail2ban tutorial |

### OpenClaw 相关

| 主题 | 链接 |
|------|------|
| OpenClaw 官方文档 | https://docs.openclaw.ai/ |
| OpenClaw GitHub | https://github.com/openclaw/openclaw |
| OpenClaw Discord | https://discord.com/invite/clawd |

---

## 紧急联系

**如果发现问题：**

1. **立即断开公网访问**
   ```bash
   # 阿里云 CLI
   aliyun ecs RevokeSecurityGroup \
     --SecurityGroupId sg-xxx \
     --NicType internet \
     --Policy Drop \
     --IpProtocol all \
     --PortRange -1/-1 \
     --SourceCidrIp 0.0.0.0/0
   ```

2. **检查日志**
   ```bash
   sudo tail -f /var/log/nginx/access.log
   sudo tail -f /var/log/auth.log
   ```

3. **封禁可疑 IP**
   ```bash
   sudo ufw deny from 可疑 IP
   ```

---

*最后更新：2026-03-09*  
*版本：v1.0*
