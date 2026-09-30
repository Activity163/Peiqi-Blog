---
title: VPS 初始化与常用一键脚本合集
date: 2026-10-01 04:00:00
categories:
  - 教程
tags:
  - VPS
  - Linux
  - 运维
  - 脚本
---

开一台新机器之后要干的事,基本就这几样:系统优化、装环境、测网络、上面板、重装、清理。这篇把常用的命令按用途整理在一起,当速查表用。

> 以下命令默认在 **Debian 12 / Ubuntu** 上执行,全部需要 root 权限。跑任何脚本之前,先确认你清楚它在干什么 —— 尤其是重装和清理类脚本。

<!-- more -->

## 一、系统优化

### 1. 开启 BBR 拥塞控制

BBR 是 Google 提出的 TCP 拥塞控制算法,在跨境、高延迟链路上对吞吐提升很明显,现在基本是标配。

```bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
lsmod | grep bbr
```

`lsmod | grep bbr` 有输出就说明模块已加载。注意这组命令是**追加写入**,重复执行会把同样的两行写进 `sysctl.conf` 多次。要重复跑的话,先确认一下:

```bash
grep -E "default_qdisc|tcp_congestion_control" /etc/sysctl.conf
```

### 2. 系统更新与基础工具

```bash
apt update && apt upgrade -y && apt install -y curl wget sudo unzip git
```

一次性把后面要用的 `curl`、`wget`、`unzip`、`git` 都装上。新机器第一件事就该跑这个。

## 二、运行环境

### 3. 安装 Python3 与 pip3

```bash
sudo apt-get install -y python3
sudo apt-get install -y python3-pip
```

### 4. 用 pip 安装第三方包

Debian 12 开始,系统 Python 默认开启 PEP 668 保护,直接 `pip3 install` 会被拒绝,需要加 `--break-system-packages` 绕过:

```bash
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple <软件包名> --break-system-packages
```

`-i` 指定清华源,国内机器装包快很多。

### 5. 一键安装 Docker

官方脚本,自动识别发行版并配置好源:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
```

装完建议顺手加个国内镜像加速,不然拉镜像会很难受。

## 三、网络测试

### 6. Speedtest 测速

```bash
sudo apt-get install curl && curl -s https://packagecloud.io/install/repositories/ookla/speedtest-cli/script.deb.sh | sudo bash && sudo apt-get install speedtest
```

### 7. iperf3 打流测试

```bash
apt install -y iperf3
```

一端 `iperf3 -s` 起服务,另一端 `iperf3 -c <IP>` 打流,用来测真实带宽上限。

### 8. NextTrace 路由追踪

比 `traceroute` 友好得多的可视化路由追踪工具:

```bash
curl -sL nxtrace.org/nt | bash
```

### 9. 流媒体解锁检测

```bash
bash <(curl -Ls IP.Check.Place)
```

检测这台机器能解锁哪些流媒体服务,以及 IP 的纯净度评分。

## 四、IPv6 与证书

### 10. 一键开启 IPv6

先找到网卡名:

```bash
ip -br link
```

再对目标网卡发起 DHCPv6 请求:

```bash
dhclient -6 <网卡名>
```

### 11. 一键申请证书

基于 acme.sh 的交互式脚本,支持签发和自动续期:

```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/Misaka-blog/acme-script/main/acme.sh && bash acme.sh
```

## 五、面板与监控

### 12. 安装哪吒监控面板(Nezha Dashboard)

```bash
wget -N https://raw.githubusercontent.com/Activity163/File/refs/heads/main/Install-Nezha-Dashboard.sh && bash Install-Nezha-Dashboard.sh
```

### 13. 安装哪吒监控客户端(Nezha Agent)

在**被监控的机器**上执行,需要填面板地址和密钥:

```bash
curl -L https://raw.githubusercontent.com/Activity163/File/refs/heads/main/Install-Nezha-Agent.sh -o Install-Nezha-Agent.sh
chmod +x Install-Nezha-Agent.sh
env NZ_SERVER=<面板地址>:443 NZ_TLS=true NZ_CLIENT_SECRET=<你的密钥> ./Install-Nezha-Agent.sh
```

`NZ_SERVER` 填面板域名加端口,`NZ_TLS=true` 表示走 HTTPS,`NZ_CLIENT_SECRET` 在面板后台添加服务器时生成。

## 六、系统重装

### 14. 一键 DD 重装系统

把当前系统整个换成指定发行版,`--username` / `--password` 是重装后的登录凭据:

```bash
wget https://raw.githubusercontent.com/bin456789/reinstall/refs/heads/main/reinstall.sh && bash reinstall.sh debian 12 --username root --password <新密码>
```

> **重装是不可逆操作**,会清空整个磁盘。执行前务必确认商家控制台里还有救援模式或重装入口,否则一旦失败就只能开工单。

## 七、系统清理

### 15. 一键清理

清理缓存、日志和残留文件,释放磁盘:

```bash
bash <(curl -sL https://raw.githubusercontent.com/leuxinovo/clearvps/main/leuql.sh)
```

## 安全提醒

整理这份清单时把两处敏感信息换成了占位符,因为原稿里写的是真实凭据:

- **重装密码**:真实 root 密码不要写进博客或任何公开仓库。
- **Nezha 密钥**:`NZ_CLIENT_SECRET` 泄露等于把监控面板的接入权限交出去。

同理,凡是带 `<...>` 的地方都是需要你按自己环境替换的参数,别原样照抄。跑第三方脚本前,建议先 `cat` 看一眼内容再执行 —— `curl | bash` 这种模式本质上是在信任脚本作者。

## 小结

按顺序走一遍,一台新机器从裸机到可用基本就齐了:先更新系统(第 2 条),再开 BBR(第 1 条),然后按需装环境、测网络、上面板。重装(第 14 条)和清理(第 15 条)属于破坏性操作,单独确认后再执行。
