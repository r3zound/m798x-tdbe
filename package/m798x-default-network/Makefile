#
# Copyright (C) 2026
#
# SPDX-License-Identifier: GPL-2.0-only
#
# m798x-default-network: 为 Tenda BE12 Pro (MT7987) baked-in 当前实际网络配置
# - LAN: br-lan = eth1 + lan4 + lan5, IP = 192.168.100.254/24
# - WAN: PPPoE on eth2, 账号 13545211473@jtkd
# - WAN0: DHCP on lan3 (第二WAN)
# - 防火墙: wan zone 包含 wan/wan6/wan0
# - DHCP: lan 从 100 开始, 限制 150 个
# - 系统: timezone Asia/Shanghai, NTP 阿里/腾讯
#

include $(TOPDIR)/rules.mk

PKG_NAME:=m798x-default-network
PKG_VERSION:=1.0
PKG_RELEASE:=1

PKG_LICENSE:=GPL-2.0-only

include $(INCLUDE_DIR)/package.mk

define Package/m798x-default-network
  SECTION:=base
  CATEGORY:=Base system
  TITLE:=m798x BE12 Pro baked-in network defaults (LAN 192.168.100.254, dual-WAN)
  PKGARCH:=all
endef

define Package/m798x-default-network/description
  This package hard-codes the current network configuration of the
  Tenda BE12 Pro router into the firmware image. It will run on first
  boot via /etc/uci-defaults/ to set:
  - LAN IP: 192.168.100.254/24 (br-lan = eth1 + lan4 + lan5)
  - WAN: PPPoE on eth2 (13545211473@jtkd)
  - WAN0: DHCP on lan3
  - Firewall: wan zone includes wan0
  - DHCP: lan starts at .100, limit 150
endef

define Build/Compile
endef

define Package/m798x-default-network/install
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_BIN) ./files/99-m798x-default-network $(1)/etc/uci-defaults/
endef

$(eval $(call BuildPackage,m798x-default-network))
