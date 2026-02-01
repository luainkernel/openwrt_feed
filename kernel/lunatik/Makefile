#
# SPDX-FileCopyrightText: (c) 2025 Ring Zero Desenvolvimento de Software LTDA
# SPDX-License-Identifier: MIT OR GPL-2.0-only
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/netfilter.mk

PKG_NAME:=lunatik
PKG_VERSION:=4.0
PKG_LICENSE:=GPL-2.0

PKG_SOURCE_URL:=https://github.com/luainkernel/lunatik
PKG_SOURCE_PROTO:=git
PKG_SOURCE_DATE:=2026-01-23
PKG_SOURCE_VERSION:=2619d9deea86e8d41be1f374946c3d4d9d54cbec

PKG_MAINTAINER:=Lourival Vieira Neto<lourival.neto@ringzero.com.br>
PKG_BUILD_PARALLEL:=1
PKG_FLAGS:=nonshared

ifeq ($(ARCH), x86)
	ifdef CONFIG_X86_32
		KLIBC_ARCH := i386
		asflags-y += -D_REGPARM
	else
		KLIBC_ARCH := x86_64
	endif
else ifeq ($(ARCH), aarch64)
	KLIBC_ARCH := arm64
else ifeq ($(ARCH), mipsel)
	KLIBC_ARCH := mips
else
	KLIBC_ARCH := $(ARCH)
endif

KLIBC_USR := klibc/usr

CONFIG_LUNATIK ?= m
CONFIG_LUNATIK_RUNTIME ?= y
CONFIG_LUNATIK_RUN ?= m

LUNATIK_MODULES := DEVICE LINUX NOTIFIER SOCKET RCU THREAD FIB DATA PROBE SYSCALL XDP FIFO XTABLE NETFILTER \
        COMPLETION CRYPTO_SHASH CRYPTO_SKCIPHER CRYPTO_AEAD CRYPTO_RNG CRYPTO_COMP CPU HID SIGNAL BYTEORDER

$(foreach c,$(LUNATIK_MODULES),\
        $(eval CONFIG_LUNATIK_$(c) ?= m))

# disable modules here, e.g.:
CONFIG_LUNATIK_PROBE := n
CONFIG_LUNATIK_SYSCALL := n
CONFIG_LUNATIK_NOTIFIER := n
CONFIG_LUNATIK_XDP := n
CONFIG_LUNATIK_XTABLE := n
CONFIG_LUNATIK_HID := n
CONFIG_LUNATIK_CPU := n

LUNATIK_CONFIG_MODULES := \
        $(foreach c,$(LUNATIK_MODULES),CONFIG_LUNATIK_$(c)=$(CONFIG_LUNATIK_$(c)))

LUNATIK_CONFIG_FLAGS := CONFIG_LUNATIK=$(CONFIG_LUNATIK) CONFIG_LUNATIK_RUNTIME=$(CONFIG_LUNATIK_RUNTIME) \
        CONFIG_LUNATIK_RUN=$(CONFIG_LUNATIK_RUN) $(LUNATIK_CONFIG_MODULES)

LUNATIK_MODULES := \
        $(foreach c,$(LUNATIK_MODULES),\
                $(if $(filter y m,$(CONFIG_LUNATIK_$(c))),$(c)))

EXTRA_KCONFIG := $(LUNATIK_CONFIG_FLAGS)

EXTRA_CFLAGS:= -Wimplicit-fallthrough=0 -Wno-declaration-after-statement \
	-I$(PKG_BUILD_DIR) -I$(PKG_BUILD_DIR)/include -I$(PKG_BUILD_DIR)/lua \
	-I$(PKG_BUILD_DIR)/$(KLIBC_USR)/include/arch/$(KLIBC_ARCH)

KBUILD_AFLAGS:= -I$(PKG_BUILD_DIR)/$(KLIBC_USR)/include/arch/$(KLIBC_ARCH)

LUA_API:= lua/lua.h lua/lauxlib.h lua/lualib.h

LUNATIK_INSTALL_PATH:= /usr/sbin

SCRIPTS_INSTALL_PATH:= /lib/modules/lua

include $(INCLUDE_DIR)/package.mk

define Package/lunatik/Default
  TITLE:=Lunatik
  URL:=$PKG_SOURCE_URL
endef

define KernelPackage/lunatik
  $(call Package/lunatik/Default)
  SUBMENU:=Other modules
  KCONFIG:=CONFIG_NETFILTER=y CONFIG_NETFILTER_ADVANCED=y
  DEPENDS:=\
	+kmod-nf-ipt \
	+lunatik-scripts \
	+kmod-crypto-rng \
	+kmod-crypto-hash \
	+kmod-crypto-hmac
  FILES:=$(PKG_BUILD_DIR)/lunatik.ko $(PKG_BUILD_DIR)/lunatik_run.ko \
	 $(PKG_BUILD_DIR)/lib/luarcu.ko $(PKG_BUILD_DIR)/lib/luadevice.ko \
	 $(PKG_BUILD_DIR)/lib/luadata.ko $(PKG_BUILD_DIR)/lib/luanetfilter.ko \
	 $(PKG_BUILD_DIR)/lib/luasocket.ko $(PKG_BUILD_DIR)/lib/luabyteorder.ko \
	 $(PKG_BUILD_DIR)/lib/luathread.ko $(PKG_BUILD_DIR)/lib/lualinux.ko \
	 $(PKG_BUILD_DIR)/lib/luafifo.ko $(PKG_BUILD_DIR)/lib/luacompletion.ko \
	 $(PKG_BUILD_DIR)/lib/luacrypto_shash.ko \
	 $(PKG_BUILD_DIR)/lib/luacrypto_skcipher.ko \
	 $(PKG_BUILD_DIR)/lib/luacrypto_aead.ko \
	 $(PKG_BUILD_DIR)/lib/luacrypto_rng.ko \
	 $(PKG_BUILD_DIR)/lib/luacrypto_comp.ko
  AUTOLOAD:=$(call AutoLoad,30,lunatik,1)
endef

define KernelPackage/lunatik/description
  Lunatik is a framework for scripting the Linux kernel with Lua.
endef

define Package/lunatik-scripts
  $(call Package/lunatik/Default)
  DEPENDS:=\
	   +lua5.4 \
	   +kmod-lunatik
endef

define Package/lunatik-scripts/description
	Lunatik kernel scripts and userspace utilities
endef

define Package/lunatik-scripts/install
	$(INSTALL_DIR) $(1)$(LUNATIK_INSTALL_PATH)
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/bin/lunatik $(1)$(LUNATIK_INSTALL_PATH)
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/lunatik
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/linux
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/socket
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/syscall
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/crypto
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/driver.lua $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/mailbox.lua $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/net.lua $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/lunatik/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/lunatik
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/socket/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/socket
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/syscall/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/syscall
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/crypto/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/crypto
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/autogen/lunatik/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/lunatik
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/autogen/linux/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/linux
endef

KERNEL_RELEASE := $(LINUX_VERSION)-$(LINUX_RELEASE)
LUNATIK_MODULES := $(strip $(LUNATIK_MODULES))

define Build/Prepare
	$(Build/Prepare/Default)
	cd $(PKG_BUILD_DIR) && CC=$(CC) ./gensymbols.sh $(LUA_API) > lunatik_sym.h
	#mkdir -p $(PKG_BUILD_DIR)/autogen/linux
	cd $(PKG_BUILD_DIR) && CC=$(CC) lua5.4 configure.lua $(KERNEL_RELEASE) $(LINUX_DIR)/include "$(LUNATIK_MODULES)"
	cp $(PKG_BUILD_DIR)/Kbuild $(PKG_BUILD_DIR)/Makefile
	$(if $(call kernel_patchver_lt,5.15),rm $(PKG_BUILD_DIR)/include/stdarg.h)
endef

define Build/Compile
	$(KERNEL_MAKE) M="$(PKG_BUILD_DIR)" \
		EXTRA_CFLAGS="$(EXTRA_CFLAGS)" \
		KBUILD_AFLAGS="$(KBUILD_AFLAGS)" \
		$(EXTRA_KCONFIG) \
		modules
endef

$(eval $(call KernelPackage,lunatik))
$(eval $(call BuildPackage,lunatik-scripts))

