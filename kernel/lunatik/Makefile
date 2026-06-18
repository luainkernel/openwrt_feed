#
# SPDX-FileCopyrightText: (c) 2025-2026 Ring Zero Desenvolvimento de Software LTDA
# SPDX-License-Identifier: MIT OR GPL-2.0-only
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/netfilter.mk

PKG_NAME:=lunatik
PKG_VERSION:=4.3.1+2026-06-18-8ac18893
PKG_LICENSE:=GPL-2.0

PKG_SOURCE_URL:=https://github.com/luainkernel/lunatik
PKG_SOURCE_PROTO:=git
PKG_SOURCE_DATE:=2026-06-18
PKG_SOURCE_VERSION:=8ac188935977938b5705e239911f2e6b1390c1dc

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

# Single source of truth for which sub-kmod bindings are built and shipped.
# Everything in LUNATIK_SUBS that is not listed here goes to CONFIG_LUNATIK_<X>=n
# and is skipped at compile time, so a `make defconfig` build (no menuconfig)
# produces exactly these .ko/.ipk plus the kmod-lunatik core.
LUNATIK_ENABLED_SUBS := byteorder completion darken data fifo linux netfilter \
	notifier set skb socket thread

# Full set of optional sub-kmods known to the feed. LUNATIK_XDP off — eBPF
# toolchain out of scope. HID off — depends on kmod-hid exports unavailable in
# minimal router kernels.
LUNATIK_SUBS := byteorder completion cpu crypto darken data fib fifo \
	linux netfilter notifier probe set signal skb socket syscall thread

EXTRA_KCONFIG := \
	CONFIG_LUNATIK=m CONFIG_LUNATIK_RUNTIME=y CONFIG_LUNATIK_RUN=m \
	CONFIG_LUNATIK_DEVICE=m CONFIG_LUNATIK_RCU=m \
	CONFIG_LUNATIK_XDP=n CONFIG_LUNATIK_HID=n \
	$(foreach s,$(LUNATIK_SUBS), \
	  CONFIG_LUNATIK_$(shell echo $(s) | tr a-z A-Z)=$(if $(filter $(s),$(LUNATIK_ENABLED_SUBS)),m,n))

# Module list passed to autogen.lua. autogen's emit.config() writes
# config.modules from it so lunatik's runtime only tries to modprobe
# bindings that were actually built.
LUNATIK_MODULES := DEVICE RCU $(LUNATIK_ENABLED_SUBS)

EXTRA_CFLAGS:= -Wimplicit-fallthrough=0 -Wno-declaration-after-statement \
	-I$(PKG_BUILD_DIR) -I$(PKG_BUILD_DIR)/include -I$(PKG_BUILD_DIR)/lua \
	-I$(PKG_BUILD_DIR)/$(KLIBC_USR)/include/arch/$(KLIBC_ARCH)

KBUILD_AFLAGS:= -I$(PKG_BUILD_DIR)/$(KLIBC_USR)/include/arch/$(KLIBC_ARCH)

LUA_API:= lua/lua.h lua/lauxlib.h lua/lualib.h

LUNATIK_INSTALL_PATH:= /usr/sbin
LUA_5.4_INSTALL_PATH := /usr/local/lib/lua/5.4
SCRIPTS_INSTALL_PATH:= /lib/modules/lua

include $(INCLUDE_DIR)/package.mk


KERNEL_RELEASE := $(LINUX_VERSION)-$(LINUX_RELEASE)

define Build/Prepare
	$(Build/Prepare/Default)
	cd $(PKG_BUILD_DIR) && CC=$(CC) ./gensymbols.sh $(LUA_API) > lunatik_sym.h
	cp $(PKG_BUILD_DIR)/Kbuild $(PKG_BUILD_DIR)/Makefile
	$(if $(call kernel_patchver_lt,5.15),rm $(PKG_BUILD_DIR)/include/stdarg.h)
endef

define Build/Compile
	cd $(PKG_BUILD_DIR) && $(KERNEL_MAKE_FLAGS) CC=$(TARGET_CC) lua5.4 autogen.lua $(LINUX_DIR) $(KERNEL_RELEASE) "$(LUNATIK_MODULES)"
	$(KERNEL_MAKE) M="$(PKG_BUILD_DIR)" \
		EXTRA_CFLAGS="$(EXTRA_CFLAGS)" \
		KBUILD_AFLAGS="$(KBUILD_AFLAGS)" \
		$(EXTRA_KCONFIG) \
		modules
	$(KERNEL_CROSS)strip --strip-debug --strip-unneeded $(PKG_BUILD_DIR)/lib/luadarken.ko
endef


# Core: kmod-lunatik

define KernelPackage/lunatik
  SUBMENU:=Lunatik
  URL:=https://github.com/luainkernel/lunatik
  TITLE:=Lunatik Lua-in-kernel runtime (core)
  DEPENDS:=+lua5.4
  DEFAULT:=y
  FILES:= \
	$(PKG_BUILD_DIR)/lunatik.ko \
	$(PKG_BUILD_DIR)/lunatik_run.ko \
	$(PKG_BUILD_DIR)/lib/luadevice.ko \
	$(PKG_BUILD_DIR)/lib/luarcu.ko
  AUTOLOAD:=$(call AutoLoad,30,lunatik luarcu luadevice lunatik_run,1)
endef

define KernelPackage/lunatik/description
  Core Lunatik runtime: the Lua interpreter, the `lunatik run` driver,
  the RCU hash table and the character device interface used by the
  `lunatik` userspace CLI. Optional bindings are in separate sub-packages.
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
endef


# Simple bindings: single .ko, no extra scripts, no external kmod deps

define KernelPackage/lunatik-byteorder
  SUBMENU:=Lunatik
  TITLE:=Lunatik byteorder bindings
  DEPENDS:=+kmod-lunatik
  DEFAULT:=y
  FILES:=$(PKG_BUILD_DIR)/lib/luabyteorder.ko
  AUTOLOAD:=$(call AutoLoad,31,luabyteorder)
endef
define KernelPackage/lunatik-byteorder/description
  Byte-order swapping primitives.
endef
$(eval $(call KernelPackage,lunatik-byteorder))


define KernelPackage/lunatik-completion
  SUBMENU:=Lunatik
  TITLE:=Lunatik completion bindings
  DEPENDS:=+kmod-lunatik
  DEFAULT:=y
  FILES:=$(PKG_BUILD_DIR)/lib/luacompletion.ko
  AUTOLOAD:=$(call AutoLoad,31,luacompletion)
endef
define KernelPackage/lunatik-completion/description
  Kernel completion synchronization primitive.
endef
$(eval $(call KernelPackage,lunatik-completion))


define KernelPackage/lunatik-cpu
  SUBMENU:=Lunatik
  TITLE:=Lunatik CPU bindings
  DEPENDS:=+kmod-lunatik
  FILES:=$(PKG_BUILD_DIR)/lib/luacpu.ko
  AUTOLOAD:=$(call AutoLoad,31,luacpu)
endef
define KernelPackage/lunatik-cpu/description
  Per-CPU statistics and topology exposure.
endef
$(eval $(call KernelPackage,lunatik-cpu))


define KernelPackage/lunatik-data
  SUBMENU:=Lunatik
  TITLE:=Lunatik data bindings
  DEPENDS:=+kmod-lunatik
  DEFAULT:=y
  FILES:=$(PKG_BUILD_DIR)/lib/luadata.ko
  AUTOLOAD:=$(call AutoLoad,31,luadata)
endef
define KernelPackage/lunatik-data/description
  Raw byte-buffer accessor used by skb and other binary protocols.
endef
$(eval $(call KernelPackage,lunatik-data))


define KernelPackage/lunatik-fib
  SUBMENU:=Lunatik
  TITLE:=Lunatik FIB bindings
  DEPENDS:=+kmod-lunatik
  FILES:=$(PKG_BUILD_DIR)/lib/luafib.ko
  AUTOLOAD:=$(call AutoLoad,31,luafib)
endef
define KernelPackage/lunatik-fib/description
  Forwarding-information-base (routing) rule bindings.
endef
$(eval $(call KernelPackage,lunatik-fib))


define KernelPackage/lunatik-fifo
  SUBMENU:=Lunatik
  TITLE:=Lunatik FIFO bindings
  DEPENDS:=+kmod-lunatik
  DEFAULT:=y
  FILES:=$(PKG_BUILD_DIR)/lib/luafifo.ko
  AUTOLOAD:=$(call AutoLoad,31,luafifo)
endef
define KernelPackage/lunatik-fifo/description
  Lockless kfifo queues usable across runtimes.
endef
$(eval $(call KernelPackage,lunatik-fifo))


define KernelPackage/lunatik-notifier
  SUBMENU:=Lunatik
  TITLE:=Lunatik notifier bindings
  DEPENDS:=+kmod-lunatik
  DEFAULT:=y
  FILES:=$(PKG_BUILD_DIR)/lib/luanotifier.ko
  AUTOLOAD:=$(call AutoLoad,31,luanotifier)
endef
define KernelPackage/lunatik-notifier/description
  Keyboard, netdevice and VT notifier-chain bindings.
endef
$(eval $(call KernelPackage,lunatik-notifier))


define KernelPackage/lunatik-probe
  SUBMENU:=Lunatik
  TITLE:=Lunatik kprobe bindings
  DEPENDS:=+kmod-lunatik
  FILES:=$(PKG_BUILD_DIR)/lib/luaprobe.ko
  AUTOLOAD:=$(call AutoLoad,31,luaprobe)
endef
define KernelPackage/lunatik-probe/description
  Kernel probes (kprobes) — requires CONFIG_KPROBES in the target kernel.
endef
$(eval $(call KernelPackage,lunatik-probe))


define KernelPackage/lunatik-set
  SUBMENU:=Lunatik
  TITLE:=Lunatik set bindings
  DEPENDS:=+kmod-lunatik
  DEFAULT:=y
  FILES:=$(PKG_BUILD_DIR)/lib/luaset.ko
  AUTOLOAD:=$(call AutoLoad,31,luaset)
endef
define KernelPackage/lunatik-set/description
  Compact immutable string set: a build-once sorted blob of keys plus an
  offset array for large lookups such as domain blocklists. Lookups are
  lockless and allocate nothing, so they are safe from softirq (netfilter,
  XDP); with a one-byte separator set:has() also matches any suffix.
endef
$(eval $(call KernelPackage,lunatik-set))


define KernelPackage/lunatik-signal
  SUBMENU:=Lunatik
  TITLE:=Lunatik signal bindings
  DEPENDS:=+kmod-lunatik
  FILES:=$(PKG_BUILD_DIR)/lib/luasignal.ko
  AUTOLOAD:=$(call AutoLoad,31,luasignal)
endef
define KernelPackage/lunatik-signal/description
  POSIX signal delivery to userspace tasks.
endef
$(eval $(call KernelPackage,lunatik-signal))


define KernelPackage/lunatik-skb
  SUBMENU:=Lunatik
  TITLE:=Lunatik skb bindings
  DEPENDS:=+kmod-lunatik +kmod-lunatik-data
  DEFAULT:=y
  FILES:=$(PKG_BUILD_DIR)/lib/luaskb.ko
  AUTOLOAD:=$(call AutoLoad,31,luaskb)
endef
define KernelPackage/lunatik-skb/description
  sk_buff accessor used by netfilter hooks and custom packet filters.
endef
$(eval $(call KernelPackage,lunatik-skb))


define KernelPackage/lunatik-thread
  SUBMENU:=Lunatik
  TITLE:=Lunatik thread bindings
  DEPENDS:=+kmod-lunatik
  DEFAULT:=y
  FILES:=$(PKG_BUILD_DIR)/lib/luathread.ko
  AUTOLOAD:=$(call AutoLoad,31,luathread)
endef
define KernelPackage/lunatik-thread/description
  Kernel threads (spawn/stop) for running Lua runtimes in the background.
endef
$(eval $(call KernelPackage,lunatik-thread))


# Bindings with their own Lua scripts

define KernelPackage/lunatik-linux
  SUBMENU:=Lunatik
  TITLE:=Lunatik linux API bindings
  DEPENDS:=+kmod-lunatik
  DEFAULT:=y
  FILES:=$(PKG_BUILD_DIR)/lib/lualinux.ko
  AUTOLOAD:=$(call AutoLoad,31,lualinux)
endef
define KernelPackage/lunatik-linux/description
  Generic kernel facilities (random, jiffies, net devices, ...) plus the
  autogenerated `linux.*` constant tables.
endef
define KernelPackage/lunatik-linux/install
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/linux
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/autogen/linux/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/linux
endef
$(eval $(call KernelPackage,lunatik-linux))


define KernelPackage/lunatik-socket
  SUBMENU:=Lunatik
  TITLE:=Lunatik socket bindings
  DEPENDS:=+kmod-lunatik
  DEFAULT:=y
  FILES:=$(PKG_BUILD_DIR)/lib/luasocket.ko
  AUTOLOAD:=$(call AutoLoad,31,luasocket)
endef
define KernelPackage/lunatik-socket/description
  Kernel socket API for Lua runtimes, including the `net` helper library.
endef
define KernelPackage/lunatik-socket/install
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/socket
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/net.lua $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/socket/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/socket
endef
$(eval $(call KernelPackage,lunatik-socket))


define KernelPackage/lunatik-syscall
  SUBMENU:=Lunatik
  TITLE:=Lunatik syscall bindings
  DEPENDS:=+kmod-lunatik
  FILES:=$(PKG_BUILD_DIR)/lib/luasyscall.ko
  AUTOLOAD:=$(call AutoLoad,31,luasyscall)
endef
define KernelPackage/lunatik-syscall/description
  System-call table lookup helpers (requires CONFIG_KPROBES).
endef
define KernelPackage/lunatik-syscall/install
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/syscall
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/syscall/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/syscall
endef
$(eval $(call KernelPackage,lunatik-syscall))


# Bindings with external kmod deps

define KernelPackage/lunatik-hid
  SUBMENU:=Lunatik
  TITLE:=Lunatik HID bindings
  DEPENDS:=+kmod-lunatik +kmod-hid
  FILES:=$(PKG_BUILD_DIR)/lib/luahid.ko
  AUTOLOAD:=$(call AutoLoad,31,luahid)
endef
define KernelPackage/lunatik-hid/description
  Human Interface Device driver registration from Lua.
endef
$(eval $(call KernelPackage,lunatik-hid))


define KernelPackage/lunatik-netfilter
  SUBMENU:=Lunatik
  TITLE:=Lunatik netfilter bindings
  DEPENDS:=+kmod-lunatik +kmod-lunatik-skb
  DEFAULT:=y
  FILES:=$(PKG_BUILD_DIR)/lib/luanetfilter.ko
  AUTOLOAD:=$(call AutoLoad,32,luanetfilter)
endef
define KernelPackage/lunatik-netfilter/description
  Netfilter hook registration. Hooks receive an skb; load order 32 so skb
  (31) is in place first.
endef
$(eval $(call KernelPackage,lunatik-netfilter))


define KernelPackage/lunatik-crypto
  SUBMENU:=Lunatik
  TITLE:=Lunatik crypto bindings
  DEPENDS:=+kmod-lunatik +kmod-crypto-aead +kmod-crypto-rng \
	+kmod-crypto-hash +kmod-crypto-hmac
  FILES:=$(PKG_BUILD_DIR)/lib/luacrypto.ko
  AUTOLOAD:=$(call AutoLoad,31,luacrypto)
endef
define KernelPackage/lunatik-crypto/description
  Linux kernel Crypto API bindings (hash, HMAC, AEAD, skcipher, RNG).
endef
define KernelPackage/lunatik-crypto/install
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/crypto
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/crypto/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/crypto
endef
$(eval $(call KernelPackage,lunatik-crypto))


define KernelPackage/lunatik-darken
  SUBMENU:=Lunatik
  TITLE:=Lunatik darken (encrypted scripts)
  DEPENDS:=+kmod-lunatik +kmod-crypto-ctr
  DEFAULT:=y
  FILES:=$(PKG_BUILD_DIR)/lib/luadarken.ko
  AUTOLOAD:=$(call AutoLoad,31,luadarken)
endef
define KernelPackage/lunatik-darken/description
  AES-256-CTR encrypted Lua script execution. The `lighten` helper
  script lives in /lib/modules/lua for encrypting payloads offline.
endef
define KernelPackage/lunatik-darken/install
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/lighten.lua $(1)$(SCRIPTS_INSTALL_PATH)/
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/util.lua $(1)$(SCRIPTS_INSTALL_PATH)/
endef
$(eval $(call KernelPackage,lunatik-darken))


$(eval $(call KernelPackage,lunatik))
