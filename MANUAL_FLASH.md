# Tenda BE12 Pro 编译固件刷机手册（双 WAN 负载均衡版）

## ⚠️ 刷机有风险，操作需谨慎

本文基于 `r3zound/m798x-tdbe` 仓库 `feature/mwan3-build` 分支编译的固件。

## 前置准备

1. **备份原固件**（强烈建议）
2. **下载编译产物**（GitHub Actions Artifacts）
3. **确认路由器已联网**（PPPoE 拨号已通）

## 步骤 1：备份原固件（路由器上操作）

SSH 登入路由器（默认 `root/admin`）：

```bash
# 备份所有 MTD 分区
mkdir -p /tmp/mtdbackup
cd /tmp/mtdbackup
for part in $(cat /proc/mtd | grep -oE 'mtd[0-9]+:' | cut -d: -f1 | sed 's/mtd//'); do
  dd if=/dev/mtd${part} of=/tmp/mtdbackup/mtd${part}.bin 2>&1
done
ls -lh /tmp/mtdbackup/
```

把这些文件下载到本地保存。

## 步骤 2：备份 OpenWrt 配置

```bash
sysupgrade -b /tmp/backup-$(date +%Y%m%d-%H%M%S).tar.gz
```

把这个文件下载到本地。

## 步骤 3：上传固件到路由器

把 `Actions` 产物里 `*tenda-be12-pro*.tar.gz` 或 `.itb` 上传到路由器 `/tmp/`：

```bash
# 用 scp 或 wget 上传
# 例如：
wget -O /tmp/firmware.tar.gz "https://your-host/firmware.tar.gz"

# 或本地电脑：
scp firmware.tar.gz root@192.168.100.254:/tmp/
```

## 步骤 4：刷机

**方式 A：保留配置（推荐，前提是配置文件兼容）**

```bash
sysupgrade -v /tmp/firmware.tar.gz
```

**方式 B：完全清除配置（最稳）**

```bash
sysupgrade -v -n /tmp/firmware.tar.gz
```

`-n` 表示不保留配置，避免新固件与旧配置冲突。

## 步骤 5：等待重启

路由器会自动重启，**断网 2-5 分钟**。等待指示灯稳定后尝试访问 192.168.100.254。

## 步骤 6：恢复配置（如果用方式 B）

按之前摸清的情况重建：
- WAN：PPPoE，账号 `13545211473@jtkd`，密码 `888888`
- LAN：192.168.100.254/24
- 其他按需

## 步骤 7：验证 mwan3 安装成功

```bash
# 查看 mwan3 版本
mwan3 --version 2>/dev/null || /usr/sbin/mwan3 --version

# 查看 iptables statistic 模块
iptables -m statistic -h 2>&1 | head -3

# 查看 LuCI 界面
# 浏览器访问 http://192.168.100.254 → 网络 → MWAN3
```

如果 `iptables -m statistic -h` 能输出一段帮助信息，说明 `kmod-ipt-ipopt` 已加载，mwan3 链路完整。

## 步骤 8：配置双 WAN 负载均衡（路由器上）

```bash
# /etc/config/network 应已包含 wan0 interface

# 8.1 创建 mwan3 接口配置
uci set mwan3.wan=interface
uci set mwan3.wan.enabled='1'
uci set mwan3.wan.proto='pppoe'
uci set mwan3.wan.family='ipv4'
uci set mwan3.wan.tracking_host='8.8.4.4'
uci set mwan3.wan.reliability='2'

uci set mwan3.wan0=interface
uci set mwan3.wan0.enabled='1'
uci set mwan3.wan0.proto='dhcp'
uci set mwan3.wan0.family='ipv4'
uci set mwan3.wan0.tracking_host='223.5.5.5'
uci set mwan3.wan0.reliability='2'

# 8.2 创建 member
uci add mwan3 member
uci set mwan3.@member[-1].interface='wan'
uci set mwan3.@member[-1].metric='1'
uci set mwan3.@member[-1].weight='1'

uci add mwan3 member
uci set mwan3.@member[-1].interface='wan0'
uci set mwan3.@member[-1].metric='2'
uci set mwan3.@member[-1].weight='1'

# 8.3 创建 policy
uci add mwan3 policy
uci set mwan3.@policy[-1].name='balanced'
uci add_list mwan3.@policy[-1].use_member='wan'
uci add_list mwan3.@policy[-1].use_member='wan0'

uci add mwan3 policy
uci set mwan3.@policy[-1].name='wan_only'
uci add_list mwan3.@policy[-1].use_member='wan'

uci add mwan3 policy
uci set mwan3.@policy[-1].name='wan0_only'
uci add_list mwan3.@policy[-1].use_member='wan0'

# 8.4 创建 rule（默认全部走 balanced）
uci add mwan3 rule
uci set mwan3.@rule[-1].name='Default'
uci set mwan3.@rule[-1].policy='balanced'
uci set mwan3.@rule[-1].dest_ip='0.0.0.0/0'
uci set mwan3.@rule[-1].proto='all'

uci commit mwan3

# 8.5 启动
/etc/init.d/mwan3 enable
/etc/init.d/mwan3 start
```

## 步骤 9：关掉 FullCone NAT（您路由器上其实没启用，确认一下）

```bash
uci set firewall.@defaults[0].fullcone='0'
uci commit firewall
/etc/init.d/firewall restart
```

## 步骤 10：验证负载均衡

```bash
# 查看 mwan3 状态
mwan3 status

# 强制从两个 WAN 各跑一次
mwan3 ifstatus wan
mwan3 ifstatus wan0

# 测试两条线路
# 在 LAN 设备上：
# mtr -r -c 5 8.8.8.8    # 看走哪条线
# ping -I 100.64.213.181 8.8.8.8  # PPPoE
# ping -I 192.168.1.51 8.8.8.8   # DHCP
```

## 🔙 回滚（如有问题）

```bash
# 如果新固件无法启动：
# 1. 保持电源 30 秒后开机，进入 failsafe 模式（启动时按 reset 键）
# 2. 或 Tenda 原厂刷机流程：访问 http://192.168.0.1 进 Tenda Web 界面刷回官方固件

# 如果新固件正常但 mwan3 配置失败：
# SSH 登入
/etc/init.d/mwan3 stop
/etc/init.d/mwan3 disable
# 单 WAN 模式继续运行
```

## 📞 排错

| 问题 | 排查方向 |
|---|---|
| 编译失败 | 看 Actions log 最后 100 行 |
| 刷机后无法启动 | Tenda 官方刷机工具刷回原厂固件 |
| 启动后无网络 | 检查 wan/wan0 接口状态、PPPoE 拨号日志 |
| mwan3 status 报错 | 检查 kmod 模块加载：`lsmod \| grep -E "ipopt\|conntrack"` |
| 负载均衡不均衡 | 这是 mwan3 的特性，不是故障；可以用 chinaroute 规则分流大陆流量 |
