# m798x-tdbe + mwan3 双 WAN 负载均衡构建分支

本分支（`feature/mwan3-build`）在 `r3zound/m798x-tdbe` 的 `mt7987_mt7992` defconfig 基础上启用了 mwan3 + 所需的内核模块，实现 **Tenda BE12 Pro 的双 WAN 负载均衡**。

## ✅ 启用的关键包

| 包 | 作用 |
|---|---|
| `mwan3` | 多 WAN 路由/负载均衡主程序 |
| `luci-app-mwan3` | LuCI 图形界面 |
| `luci-i18n-mwan3-zh-cn` | 中文界面 |
| `luci-app-mwan3helper-chinaroute` | 中国大陆 IP 分流（可选） |
| `ipset` / `libipset` | IP 集合支持 |
| `ip-full` | 替换 ip-tiny（提供 ip rule / ip route 等命令） |
| `ip-bridge` | bridge 管理命令 |
| `kmod-ipt-conntrack` | conntrack 基础（mwan3 必需） |
| `kmod-ipt-conntrack-extra` | connmark / recent / connbytes |
| `kmod-ipt-extra` | addrtype / owner / condition |
| `kmod-ipt-ipopt` | CLASSIFY target / DSCP / statistic |
| `kmod-ipt-iprange` | iprange 模块 |
| `kmod-ipt-ipset` | ipset 内核模块 |
| `kmod-ipt-nat` | NAT 基础 |
| `kmod-ipt-physdev` | bridge 物理设备匹配 |
| `iptables-mod-conntrack-extra` | 用户态 iptables 扩展（配套） |
| `iptables-mod-extra` | 用户态 iptables 扩展（配套） |
| `iptables-mod-filter` | string match |
| `iptables-mod-hashlimit` | 速率限制 |
| `iptables-mod-ipopt` | 用户态 iptables 扩展（配套） |

## ⚠️ 保留的现有配置

- `luci-app-passwall` / `argon` / `eqos-mtk` / `upnpd` —— 全部保留
- `MTK_HNAT` 硬件加速 —— 保留
- `MT7992` 闭源 Wi-Fi 驱动 —— 保留
- `fullconenat` —— **保持 disabled**（用户要求关掉 FullCone NAT）

## 🚀 触发编译

GitHub Actions → `Build Tenda BE12 Pro with mwan3` → Run workflow → 选择 `mt7987_mt7992_mwan3` → 启动

预计耗时：**90-150 分钟**（首次）或 **30-60 分钟**（有缓存）

## 📦 编译产物

Actions Artifacts → `tenda-be12-pro-firmware-mt7987_mt7992_mwan3`，保留 30 天

产物包括：
- `*tenda-be12-pro*.tar.gz`（包含 rootfs + kernel，适合 sysupgrade）
- `*tenda-be12-pro*.itb`（FIT 镜像，适合某些场景）

## 🛠️ 刷机步骤

详见 `MANUAL_FLASH.md`（同级目录）。

## 📞 反馈 / 排错

如果编译失败，请提供 Actions build log 最后 100 行（已在 workflow 里自动打印）。
