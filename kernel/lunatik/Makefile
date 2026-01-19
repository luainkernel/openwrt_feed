#
# SPDX-FileCopyrightText: (c) 2025 Ring Zero Desenvolvimento de Software LTDA
# SPDX-License-Identifier: MIT OR GPL-2.0-only
#

include $(TOPDIR)/rules.mk
-include $(TOPDIR)/.config
include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/netfilter.mk

PKG_NAME:=lunatik
PKG_VERSION:=3.6.4
PKG_LICENSE:=GPL-2.0

PKG_SOURCE_URL:=https://github.com/Zapper9982/lunatik
PKG_SOURCE_PROTO:=git
PKG_SOURCE_DATE:=2026-01-19
PKG_SOURCE_VERSION:=4e48ccba6f2a92b1a4f90a69a1ea4f605752e694

PKG_MAINTAINER:=Lourival Vieira Neto<lourival.neto@gmail.com>
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

EXTRA_KCONFIG= CONFIG_LUNATIK=m \
	CONFIG_LUNATIK_RUNTIME=y \
	CONFIG_LUNATIK_RUN=$(if $(CONFIG_PACKAGE_kmod-lunatik-run),m,n) \
	CONFIG_LUNATIK_DEVICE=$(if $(CONFIG_PACKAGE_kmod-lunatik-device),m,n) \
	CONFIG_LUNATIK_LINUX=$(if $(CONFIG_PACKAGE_kmod-lunatik-linux),m,n) \
	CONFIG_LUNATIK_SOCKET=$(if $(CONFIG_PACKAGE_kmod-lunatik-socket),m,n) \
	CONFIG_LUNATIK_RCU=$(if $(CONFIG_PACKAGE_kmod-lunatik-rcu),m,n) \
	CONFIG_LUNATIK_THREAD=$(if $(CONFIG_PACKAGE_kmod-lunatik-thread),m,n) \
	CONFIG_LUNATIK_FIB=$(if $(CONFIG_PACKAGE_kmod-lunatik-fib),m,n) \
	CONFIG_LUNATIK_DATA=$(if $(CONFIG_PACKAGE_kmod-lunatik-data),m,n) \
	CONFIG_LUNATIK_PROBE=$(if $(CONFIG_PACKAGE_kmod-lunatik-probe),m,n) \
	CONFIG_LUNATIK_SYSCALL=$(if $(CONFIG_PACKAGE_kmod-lunatik-syscall),m,n) \
	CONFIG_LUNATIK_XDP=$(if $(CONFIG_PACKAGE_kmod-lunatik-xdp),m,n) \
	CONFIG_LUNATIK_FIFO=$(if $(CONFIG_PACKAGE_kmod-lunatik-fifo),m,n) \
	CONFIG_LUNATIK_XTABLE=$(if $(CONFIG_PACKAGE_kmod-lunatik-xtable),m,n) \
	CONFIG_LUNATIK_NETFILTER=$(if $(CONFIG_PACKAGE_kmod-lunatik-netfilter),m,n) \
	CONFIG_LUNATIK_COMPLETION=$(if $(CONFIG_PACKAGE_kmod-lunatik-completion),m,n) \
	CONFIG_LUNATIK_CRYPTO_SHASH=$(if $(CONFIG_PACKAGE_kmod-lunatik-crypto-shash),m,n) \
	CONFIG_LUNATIK_CRYPTO_SKCIPHER=$(if $(CONFIG_PACKAGE_kmod-lunatik-crypto-skcipher),m,n) \
	CONFIG_LUNATIK_CRYPTO_AEAD=$(if $(CONFIG_PACKAGE_kmod-lunatik-crypto-aead),m,n) \
	CONFIG_LUNATIK_CRYPTO_RNG=$(if $(CONFIG_PACKAGE_kmod-lunatik-crypto-rng),m,n) \
	CONFIG_LUNATIK_CRYPTO_COMP=$(if $(CONFIG_PACKAGE_kmod-lunatik-crypto-comp),m,n) \
	CONFIG_LUNATIK_CPU=$(if $(CONFIG_PACKAGE_kmod-lunatik-cpu),m,n) \
	CONFIG_LUNATIK_HID=$(if $(CONFIG_PACKAGE_kmod-lunatik-hid),m,n) \
	CONFIG_LUNATIK_SIGNAL=$(if $(CONFIG_PACKAGE_kmod-lunatik-signal),m,n) \
	CONFIG_LUNATIK_BYTEORDER=$(if $(CONFIG_PACKAGE_kmod-lunatik-byteorder),m,n) \
	CONFIG_LUNATIK_NOTIFIER=$(if $(CONFIG_PACKAGE_kmod-lunatik-notifier),m,n) \
	CONFIG_LUNATIK_INSTALL_EXAMPLES=$(if $(CONFIG_PACKAGE_lunatik-examples),y,n) \
	CONFIG_LUNATIK_INSTALL_TESTS=$(if $(CONFIG_PACKAGE_lunatik-tests),y,n)

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
  DEPENDS:=+lunatik-scripts
endef

define KernelPackage/lunatik
  SUBMENU:=Other modules
  TITLE:=Lunatik Lua Interpreter
  URL:=$(PKG_SOURCE_URL)
  DEPENDS:=+lunatik-scripts
  FILES:=$(PKG_BUILD_DIR)/lunatik.ko
  AUTOLOAD:=$(call AutoLoad,30,lunatik,1)
endef

define KernelPackage/lunatik/description
  Lunatik is a framework for scripting the Linux kernel with Lua.
endef

define LunatikModule
  define KernelPackage/lunatik-$(1)
    SUBMENU:=Other modules
    TITLE+= $(2)
    DEPENDS+= +kmod-lunatik $(3)
    FILES:=$(PKG_BUILD_DIR)/$(4)
    AUTOLOAD:=$(call AutoLoad,31,lunatik-$(1),1)
    URL:=$(PKG_SOURCE_URL)
  endef
  $$(eval $$(call KernelPackage,lunatik-$(1)))
endef


define Package/lunatik-scripts
  $(call Package/lunatik/Default)
  DEPENDS:=+lua5.4
endef

define Package/lunatik-scripts/description
	Lunatik kernel scripts and userspace utilities
endef

define Package/lunatik-scripts/install
	$(INSTALL_DIR) $(1)$(LUNATIK_INSTALL_PATH)
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/bin/lunatik $(1)$(LUNATIK_INSTALL_PATH)
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/lunatik
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/socket
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/syscall
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/driver.lua $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/mailbox.lua $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/net.lua $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/lunatik/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/lunatik
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/socket/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/socket
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/syscall/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/syscall
endef

define Package/lunatik-examples
  $(call Package/lunatik/Default)
  TITLE+= Examples
  DEPENDS+= +lunatik-scripts
endef

define Package/lunatik-examples/description
  Lunatik example scripts
endef

define Package/lunatik-examples/install
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/examples
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/examples/echod
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/examples/filter
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/examples/dnsblock
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/examples/dnsdoctor
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/examples/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/examples
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/examples/echod/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/examples/echod
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/examples/filter/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/examples/filter
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/examples/dnsblock/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/examples/dnsblock
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/examples/dnsdoctor/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/examples/dnsdoctor
endef

define Package/lunatik-tests
  $(call Package/lunatik/Default)
  TITLE+= Tests
  DEPENDS+= +lunatik-scripts
endef

define Package/lunatik-tests/description
  Lunatik test scripts
endef

define Package/lunatik-tests/install
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/tests
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/tests/rcumap_sync
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/tests/crypto
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/tests/rcumap_sync/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/tests/rcumap_sync
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/tests/crypto/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/tests/crypto
endef

define Build/Prepare
	$(Build/Prepare/Default)
	cd $(PKG_BUILD_DIR) && CC=gcc ./gensymbols.sh $(LUA_API) > lunatik_sym.h
	cp $(PKG_BUILD_DIR)/Kbuild $(PKG_BUILD_DIR)/Makefile
	$(if $(call kernel_patchver_lt,5.15),rm $(PKG_BUILD_DIR)/include/stdarg.h)
endef

define Build/Compile
	$(info LINUX_DIR=$(LINUX_DIR))
	$(MAKE) -C $(LINUX_DIR) \
		M="$(PKG_BUILD_DIR)" \
		EXTRA_CFLAGS="$(EXTRA_CFLAGS)" \
		KBUILD_AFLAGS="$(KBUILD_AFLAGS)" \
		$(KERNEL_MAKE_FLAGS) \
		$(EXTRA_KCONFIG) \
		modules
endef

$(eval $(call KernelPackage,lunatik))
$(eval $(call BuildPackage,lunatik-scripts))
$(eval $(call BuildPackage,lunatik-examples))
$(eval $(call BuildPackage,lunatik-tests))

$(eval $(call LunatikModule,run,Run (Execution Support),,lunatik_run.ko))
$(eval $(call LunatikModule,device,Device Support,,lib/luadevice.ko))
$(eval $(call LunatikModule,linux,Linux API Bindings,,lib/lualinux.ko))
$(eval $(call LunatikModule,socket,Socket Support,,lib/luasocket.ko))
$(eval $(call LunatikModule,rcu,RCU Support,,lib/luarcu.ko))
$(eval $(call LunatikModule,thread,Thread Support,,lib/luathread.ko))
$(eval $(call LunatikModule,fib,FIB Support,,lib/luafib.ko))
$(eval $(call LunatikModule,data,Data Support,,lib/luadata.ko))
$(eval $(call LunatikModule,probe,Probe Support,,lib/luaprobe.ko))
$(eval $(call LunatikModule,syscall,Syscall Support,,lib/luasyscall.ko))
$(eval $(call LunatikModule,xdp,XDP Support,,lib/luaxdp.ko))
$(eval $(call LunatikModule,fifo,FIFO Support,,lib/luafifo.ko))
$(eval $(call LunatikModule,xtable,Xtable Support,+kmod-nf-ipt,lib/luaxtable.ko))
$(eval $(call LunatikModule,netfilter,Netfilter bindings,,lib/luanetfilter.ko))
$(eval $(call LunatikModule,completion,Completion Support,,lib/luacompletion.ko))
$(eval $(call LunatikModule,crypto-shash,Crypto SHASH Support,,lib/luacrypto_shash.ko))
$(eval $(call LunatikModule,crypto-skcipher,Crypto SKCIPHER Support,,lib/luacrypto_skcipher.ko))
$(eval $(call LunatikModule,crypto-aead,Crypto AEAD Support,,lib/luacrypto_aead.ko))
$(eval $(call LunatikModule,crypto-rng,Crypto RNG Support,,lib/luacrypto_rng.ko))
$(eval $(call LunatikModule,crypto-comp,Crypto COMP Support,,lib/luacrypto_comp.ko))
$(eval $(call LunatikModule,cpu,CPU Support,,lib/luacpu.ko))
$(eval $(call LunatikModule,hid,HID Support,,lib/luahid.ko))
$(eval $(call LunatikModule,signal,Signal Support,,lib/luasignal.ko))
$(eval $(call LunatikModule,byteorder,Byteorder Support,,lib/luabyteorder.ko))
$(eval $(call LunatikModule,notifier,Notifier Support,,lib/luanotifier.ko))

