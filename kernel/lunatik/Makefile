#
# SPDX-FileCopyrightText: (c) 2025-2026 Ring Zero Desenvolvimento de Software LTDA
# SPDX-License-Identifier: MIT OR GPL-2.0-only
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/netfilter.mk

PKG_NAME:=lunatik
PKG_VERSION:=4.2
PKG_LICENSE:=GPL-2.0

PKG_SOURCE_URL:=https://github.com/luainkernel/lunatik
PKG_SOURCE_PROTO:=git
PKG_SOURCE_DATE:=2026-03-24
PKG_SOURCE_VERSION:=03003c08f929f77ce76bc913966891bd760eb613

PKG_MAINTAINER:=Marcel Moura<marcel.moura@ringzero.com.br>
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

# keep module order in sync with the main lunatik Makefile
LUNATIK_MODULES := DEVICE LINUX NOTIFIER SOCKET RCU THREAD FIB DATA PROBE SYSCALL XDP FIFO SKB NETFILTER \
        COMPLETION CRYPTO CPU HID SIGNAL BYTEORDER DARKEN

LUNATIK_MODULE_FILENAME = lua$(shell echo $(1) | tr '[:upper:]' '[:lower:]'))

LUNATIK_MODULES := $(foreach c,$(LUNATIK_MODULES), \
    $(if $(CONFIG_LUNATIK_$(c)),$(c)))

LUNATIK_CONFIG_MODULES := $(foreach c,$(LUNATIK_MODULES),CONFIG_LUNATIK_$(c)=m)

EXTRA_KCONFIG := CONFIG_LUNATIK=$(if $(CONFIG_PACKAGE_kmod-lunatik),m,n) \
	CONFIG_LUNATIK_RUNTIME=$(if $(CONFIG_LUNATIK_RUNTIME),y,n) \
	CONFIG_LUNATIK_RUN=$(if $(CONFIG_LUNATIK_RUN),m,n) \
	$(LUNATIK_CONFIG_MODULES)

EXTRA_CFLAGS:= -Wimplicit-fallthrough=0 -Wno-declaration-after-statement \
	-I$(PKG_BUILD_DIR) -I$(PKG_BUILD_DIR)/include -I$(PKG_BUILD_DIR)/lua \
	-I$(PKG_BUILD_DIR)/$(KLIBC_USR)/include/arch/$(KLIBC_ARCH)

KBUILD_AFLAGS:= -I$(PKG_BUILD_DIR)/$(KLIBC_USR)/include/arch/$(KLIBC_ARCH)

LUA_API:= lua/lua.h lua/lauxlib.h lua/lualib.h

LUNATIK_INSTALL_PATH:= /usr/sbin
LUA_5.4_INSTALL_PATH := /usr/local/lib/lua/5.4
SCRIPTS_INSTALL_PATH:= /lib/modules/lua

include $(INCLUDE_DIR)/package.mk

define KernelPackage/lunatik
  SUBMENU:=Other modules
  URL:=$PKG_SOURCE_URL
  TITLE:=Lunatik Lua Interpreter
  DEPENDS:=+lua5.4 \
	+LUNATIK_CRYPTO:kmod-crypto-aead \
	+LUNATIK_CRYPTO:kmod-crypto-rng \
	+LUNATIK_CRYPTO:kmod-crypto-hash \
	+LUNATIK_CRYPTO:kmod-crypto-hmac
  FILES:=$(PKG_BUILD_DIR)/lunatik.ko \
	 $(if $(CONFIG_LUNATIK_RUN),$(PKG_BUILD_DIR)/lunatik_run.ko) \
	 $(foreach mod,$(LUNATIK_MODULES),$(PKG_BUILD_DIR)/lib/$(call LUNATIK_MODULE_FILENAME,$(mod)).ko)
  AUTOLOAD:=$(call AutoLoad,30, \
	lunatik \
	$(if $(filter RCU,$(LUNATIK_MODULES)),luarcu) \
	$(if $(filter DEVICE,$(LUNATIK_MODULES)),luadevice) \
	$(if $(CONFIG_LUNATIK_RUN),lunatik_run) \
	$(foreach mod,$(filter-out RCU DEVICE,$(LUNATIK_MODULES)),$(call LUNATIK_MODULE_FILENAME,$(mod))),1)
endef

define KernelPackage/lunatik/description
  Lunatik is a framework for scripting the Linux kernel with Lua.
endef

define KernelPackage/lunatik/config
	source "$(SOURCE)/Config.in"
endef

define KernelPackage/lunatik/install
	$(INSTALL_DIR) $(1)$(LUNATIK_INSTALL_PATH)
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/bin/lunatik $(1)$(LUNATIK_INSTALL_PATH)
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/lunatik
	$(INSTALL_DIR) $(1)$(LUA_5.4_INSTALL_PATH)/lunatik

	$(INSTALL_DATA) $(PKG_BUILD_DIR)/driver.lua $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/mailbox.lua $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/lunatik/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/lunatik
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/autogen/lunatik/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/lunatik
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/autogen/lunatik/config.lua $(1)$(LUA_5.4_INSTALL_PATH)/lunatik

	$(if $(CONFIG_LUNATIK_CRYPTO), \
		$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/crypto && \
		$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/crypto/hkdf.lua $(1)$(SCRIPTS_INSTALL_PATH)/crypto)
	$(if $(CONFIG_LUNATIK_LINUX), \
		$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/linux && \
		$(INSTALL_DATA) $(PKG_BUILD_DIR)/autogen/linux/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/linux)
	$(if $(CONFIG_LUNATIK_SOCKET), \
		$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/socket && \
		$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/net.lua $(1)$(SCRIPTS_INSTALL_PATH) && \
		$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/socket/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/socket)
	$(if $(CONFIG_LUNATIK_SYSCALL), \
		$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/syscall \
		$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/syscall/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/syscall)
	$(if $(CONFIG_LUNATIK_DARKEN), \
		$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/lighten.lua $(1)$(SCRIPTS_INSTALL_PATH) && \
		$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/util.lua $(1)$(SCRIPTS_INSTALL_PATH))
endef

KERNEL_RELEASE := $(LINUX_VERSION)-$(LINUX_RELEASE)
LUNATIK_MODULES := $(strip $(LUNATIK_MODULES))

define Build/Prepare
	$(Build/Prepare/Default)
	cd $(PKG_BUILD_DIR) && CC=$(CC) ./gensymbols.sh $(LUA_API) > lunatik_sym.h
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
	$(KERNEL_CROSS)strip --strip-debug --strip-unneeded $(PKG_BUILD_DIR)/lib/luadarken.ko
endef

$(eval $(call KernelPackage,lunatik))

