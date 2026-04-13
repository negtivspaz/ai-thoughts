# OpenClaw 安全风险：给用户与自托管者的警钟

![secImg](img/2603271113_openclaw-security.png)

OpenClaw 🦞 是一款流行的开源 AI 代理平台，提供强大的自动化能力，但近来的 Reddit 讨论与安全审计揭示了严重的安全隐患。本篇文章梳理了已曝光的风险、真实事件与可行修复建议，帮助你在自托管时采取保护措施，或决定是否迁移到更安全的替代方案。

## 核心漏洞概览

OpenClaw 的设计赋予代理广泛的系统访问权限，而社区贡献的 "skills"（技能）通过 ClawHub 分发，这放大了供应链风险。多次审计显示：在数千个技能中，有相当比例存在安全问题，部分甚至含有恶意代码（如下载器、键盘记录器、数据外发与命令注入器）。这些技能往往在被移除后又重新出现，规避了简单的人工审查。

默认部署常将端口（如 8000、18789 等）暴露到公网。扫描数据显示有数十万实例在线，且部分实例泄露配置或数据库。输入来源（邮件、Markdown、网页内容）易被嵌入可执行负载并被以近似 root 的权限处理。针对这些问题，Kaspersky、独立研究员与社区审计均发现实证案例，建议将 OpenClaw 视为高风险服务并强制隔离运行。

## 真实事件与攻击链

社区与安全博客汇总出的时间线显示多起可验证的事件：恶意技能传播、配置泄露、以及因默认配置暴露的远程执行漏洞。

- ClawHub 恶意活动（2026 年 1–2 月）

  多达数十到数百款技能被发现包含编码掩盖的恶意负载、伪造依赖与不加密的 HTTP 外发。一些样例会在 Windows/macOS/Linux 上部署加密勒索或凭据窃取工具。此类技能常被重新上传并以微小改动规避拦截。

  推荐检查流程（Linux/macOS）：

  ```bash
  # Clone skills repo for audit
  git clone https://github.com/openclaw/skills ~/openclaw-skills
  cd ~/openclaw-skills

  # Grep for common malicious patterns (exfil, downloads, encoding)
  grep -r -i "curl.*http\|wget.*http\|base64\|powershell\|cmd.exe\|keylog" . --include="*.json" --include="*.yaml"

  # Check for non-HTTPS endpoints
  grep -r -i "http://" . | grep -v "https"

  # Remove suspicious skills
  find . -name "*research*" -o -name "*hsk*" | xargs rm -rf

  # Reinstall clean: backup first
  cp -r ~/openclaw-skills ~/openclaw-skills.bak
  ```

- Moltbook 泄露案例（2026 年 2 月）

  某流行技能通过 Discord webhook 将用户配置（含 API key、主机 IP）外发，影响数万实例并在暗网出现数据样本。检测到此类外发的常用命令包括在系统日志中查找 webhook/HTTP POST 行为并扫描运行进程。

  ```bash
  # Scan local OpenClaw logs for exfil (Discord, external APIs)
  sudo journalctl -u openclaw | grep -i "discord\|webhook\|http.*post" | tail -20

  # Check running processes for anomalies
  ps aux | grep openclaw | grep -E "curl|wget|nc"
  kill -9 <PID>  # Kill suspect process

  # Inspect config for leaks
  grep -r "api_key\|token\|webhook" ~/.openclaw/config.yaml
  ```

- 端口暴露与远程执行（持续性问题）

  大规模扫描表明大量实例在默认端口上被暴露，而某些演示（如利用 Zenity 的示例）展示了通过 WebSocket 或用户输入触发的零点击远程命令执行。攻击者利用这些默认暴露的服务自动化传播恶意技能或执行载荷。

  快速阻断示例：

  ```bash
  # On Ubuntu/Debian (Linux)
  sudo netstat -tuln | grep 8000  # Confirm exposure
  sudo ufw deny 8000  # Block port
  sudo systemctl restart openclaw

  # macOS (use pfctl or firewall)
  sudo pfctl -f /etc/pf.conf  # Edit to block 8000/tcp
  sudo pfctl -E
  ```

## 分级加固清单（实操优先）

下面按三个层级给出可执行的硬化建议，适用于 Linux/macOS 环境。先在 VM 中测试再上线。

### Tier 1：基础防护（必做）

- 关闭 root 登录并启用密钥认证
- 启用主机防火墙（UFW）与 fail2ban，默认拒绝传入流量，仅允许可信 IP 的 SSH
- 将 OpenClaw 组件绑定到 loopback（127.0.0.1），禁止任意网卡监听

示例：

```bash
# Disable root SSH, enforce keys (Linux)
sudo sed -i 's/#PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/#PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# Install fail2ban/UFW
sudo apt update && sudo apt install ufw fail2ban -y
sudo ufw enable; sudo ufw default deny incoming
sudo ufw allow ssh  # From your IP only: sudo ufw allow from <YOUR_IP> to any port 22
```

### Tier 2：容器隔离（强烈推荐）

使用 Docker 以最小化主机暴露，限制网络访问与挂载卷。Simon Willison 等安全社区成员建议只在隔离环境中运行非托管代理。

```bash
# Pull official image (verify tag)
docker pull openclaw/openclaw:latest

# Run isolated (no ports, volumes minimal)
docker run -d --name secure-claw \
  --network none \  # No net access
  -v /path/to/safe/config:/config:ro \
  -v /tmp/claw-data:/data \
  openclaw/openclaw:latest

# Inspect container
docker logs secure-claw | grep -i "skill\|load"
docker exec secure-claw ps aux  # Check processes
```

### Tier 3：高级隔离与零信任（企业/多租户）

- 使用 Tailscale/VPN 保持无公网端口访问
- 在专用 VLAN / VM 中运行代理，限制能接触到的网络资源
- 采用技能白名单策略，仅允许经过审计的技能加载

示例（Tailscale + 白名单）：

```bash
# Tailscale VPN (no public ports)
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --authkey=<your-key>

# Whitelist skills
nano ~/.openclaw/allowlist.yaml  # Only vetted: - safe-skill-1
openclaw --config ~/.openclaw/config.yaml --skills allowlist.yaml

# Audit runtime
sudo strace -p $(pgrep openclaw) -e trace=execve  # Trace execs
```

额外建议：关闭不必要的工具，启用 unattended-upgrades，定期用静态/动态分析工具审计技能目录。

## 网关（Gateway）与端口上下文 — 为什么 18789/8000 很危险

- Gateway（默认端口 18789）提供 WebSocket 通道与控制面板，若绑定到 0.0.0.0 则会对公网可见。
- Vector/API（常见为 8000）也可能暴露敏感操作或配置。

正确做法是将这些服务绑定到 loopback：

```bash
openclaw config set gateway.bind "loopback"
openclaw config set vector.bind "loopback"
openclaw config set frontend.bind "loopback"

# Verify
openclaw config get gateway.bind
openclaw gateway status
sudo netstat -tuln | grep -E "18789|8000|3000"
```

并在需要远程访问时使用 SSH 端口转发或 Tailscale，而不是直接暴露端口。

## 替代方案与结论

如果你需要可审计、低风险的代理或自动化：

- 优先考虑受管理的云 API（例如企业级 LLM 服务）或经过审计的本地框架（LangChain 的沙箱化部署、经审计的 Auto-GPT Forks）。
- 若必须自托管，请默认采用 Docker + 私有网络 + 严格白名单策略；把所有技能视为潜在恶意代码。

总结：OpenClaw 的易用性伴随着显著的安全成本。大量曝光的实例说明了默认配置与开放生态的危险性。立即审计、隔离并在可行时迁移，是避免被动成为下一个受害者的唯一合理策略。

---

## References
- [OpenClaw security worse than expected](https://www.reddit.com/r/AI_Agents/comments/1r3u98p/openclaw_security_is_worse_than_i_expected_and_im/)
- [Selfhosting security minefield](https://www.reddit.com/r/selfhosted/comments/1qwn5i9/selfhosting_openclaw_is_a_security_minefield/)
- [Every vulnerability documented](https://www.reddit.com/r/LocalLLaMA/comments/1r81vw2/every_openclaw_security_vulnerability_documented/)
- [18k instances scanned](https://www.reddit.com/r/MachineLearning/comments/1r30nzv/d_we_scanned_18000_exposed_openclaw_instances_and/)
- [Malicious skill issue](https://github.com/openclaw/openclaw/issues/37664)

tag: #opensource #linux #openclaw #ai #clawhub #skill #security
